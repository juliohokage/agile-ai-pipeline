# Phase 5: Story Validation

## When This Runs
Always — after Phase 4 (story writing).

## Agent
**Kai** (Story Validator) — `.claude/agents/story-validator.md`

## Input
- All `docs/stories/EPIC-*-stories.md` files (or subset for focused runs)
- `docs/EPICS.md` (for gap coverage check)

## Process

### Step 1: Determine validation mode

| Pipeline Mode | Validation Mode |
|--------------|----------------|
| Discovery | Full validation (all story files) |
| Focused | Subset validation (only new/modified Epic story files) |
| Re-validation (after fixes) | Re-validation (only previously failed stories) |

### Step 2: Dispatch Story Validator

Launch the `story-validator` subagent via the Agent tool with prompt:

**Full mode:**
"Run in FULL VALIDATION mode. Read all docs/stories/EPIC-*-stories.md files.
Create docs/VALIDATION_REPORT.md."

**Subset mode:**
"Run in SUBSET VALIDATION mode. Validate only: {list of Epic story files}.
Read existing docs/VALIDATION_REPORT.md if it exists. Update/append results for
the specified Epics. Preserve results for other Epics. Recalculate overall status."

**Re-validation mode:**
"Run in RE-VALIDATION mode. Read existing docs/VALIDATION_REPORT.md. Only re-check
stories that previously FAILed or had WARNINGs. Update their status. Recalculate totals."

### Step 3: Verify output

- Confirm `docs/VALIDATION_REPORT.md` was created/updated
- Check if "Ready for Jira: YES" is in the report

### Step 4: Update state

Update `docs/.pipeline-state.json`:
- If PASS: `validation.status` → "PASS"
- If FAIL: `validation.status` → "FAIL"
- `validation.timestamp` → current ISO-8601
- Compute `validation.stories_hash`:
  `find docs/stories -name "EPIC-*-stories.md" -exec sha256sum {} \; | sort | sha256sum`
- Update `ids.highest_story` with highest story number per Epic

### Step 5: Handle failures

If validation FAILS:
1. Report the failures to the user
2. Ask: "Validation found {n} issues. Would you like me to:
   (a) Auto-fix — re-run the story writer for affected Epics with the feedback
   (b) Manual fix — you fix the issues yourself, then run `/run-pipeline phase:validate`"
3. If auto-fix:
   - Re-dispatch story writers for affected Epics with the validation feedback injected
   - Re-run validation (re-validation mode)
   - Repeat until PASS or user chooses manual fix

## Output
- `docs/VALIDATION_REPORT.md` — pass/fail per story, cross-story issues, "Ready for Jira" gate

## State Updates
```json
{
  "validation": {
    "status": "{PASS|FAIL}",
    "timestamp": "{ISO-8601}",
    "stories_hash": "{sha256}"
  },
  "ids": {
    "highest_story": { "{epic}": {n} }
  },
  "checkpoint": {
    "current_phase": "validate"
  }
}
```

## Checkpoint
None at Phase 5 itself — the checkpoint comes after Phase 5.5 (adversarial review).
In lite mode (no Phase 5.5), the checkpoint happens here instead:

### lite mode checkpoint:
Set `checkpoint.awaiting_human` to `true`.

"Validation {PASSED|FAILED}. {summary}.
Review `docs/VALIDATION_REPORT.md`.
Reply 'create-jira' to push to Jira, 'skip' to stop here, or provide feedback."

After user responds: set `checkpoint.awaiting_human` to `false`.
If user says 'skip': stop the pipeline. All local work is complete.
