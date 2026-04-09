# Phase 5.5: Adversarial Review + Edge Case Hunting

## When This Runs
- **Full mode**: Adversarial review only (Rex)
- **Thorough mode**: Adversarial review (Rex) + Edge case hunter (Nova)
- **Lite mode**: Skip entirely

## Agent
- **Rex** (Adversarial Reviewer) — `.claude/agents/adversarial-reviewer.md` — runs in full + thorough
- **Nova** (Edge Case Hunter) — `.claude/agents/edge-case-hunter.md` — runs in thorough only

## Input
- All `docs/stories/EPIC-*-stories.md` files
- `docs/GAP_ANALYSIS.md` (for Rex — gap coverage check)
- `docs/VALIDATION_REPORT.md` (to append findings)

## Process

### Step 1: Dispatch Adversarial Reviewer (full + thorough)

Launch the `adversarial-reviewer` subagent via the Agent tool:

"You are Rex. Review all story files in docs/stories/ against docs/GAP_ANALYSIS.md.
Find completeness gaps, hidden assumptions, vague requirements, missing error scenarios,
and contradictions. You must find minimum 10 issues.
Append your findings to docs/VALIDATION_REPORT.md under '### Adversarial Review Findings'."

### Step 2: Dispatch Edge Case Hunter (thorough only)

If mode is thorough, launch the `edge-case-hunter` subagent:

"You are Nova. Read all Gherkin acceptance criteria from docs/stories/EPIC-*-stories.md.
Enumerate every execution path. Report only unhandled paths.
Append your findings to docs/VALIDATION_REPORT.md under '### Edge Case Analysis'."

In thorough mode, Rex and Nova MUST NOT write to the same file in parallel.
Instead, dispatch them using `superpowers:dispatching-parallel-agents` but have each
agent return findings as structured output (not append to a file). After both complete,
the orchestrator writes a single combined update to `docs/VALIDATION_REPORT.md`.

### Step 3: Verify output

- Confirm `docs/VALIDATION_REPORT.md` was updated with new sections
- Count findings: adversarial review issues + edge case unhandled paths

### Step 4: Pre-Jira verification

Use `superpowers:verification-before-completion` to run pre-flight checks:
1. `docs/.pipeline-state.json` → `validation.status` is "PASS" or "PASS_WITH_WARNINGS"
2. Stories hash matches (re-compute and compare)
3. Review findings are recorded in VALIDATION_REPORT.md

### Step 5: Update state

Update `docs/.pipeline-state.json`:
- `checkpoint.current_phase` → "review"
- `checkpoint.awaiting_human` → `true`

## Output
- Updated `docs/VALIDATION_REPORT.md` with:
  - `### Adversarial Review Findings` (full + thorough)
  - `### Edge Case Analysis` (thorough only)

## State Updates
```json
{
  "checkpoint": {
    "current_phase": "review",
    "awaiting_human": true
  }
}
```

## Checkpoint

**HUMAN CHECKPOINT** — mode-dependent:

### full mode:
"Validation PASSED. Adversarial review found {n} issues ({critical} critical, {high} high).
Review `docs/VALIDATION_REPORT.md`.
Reply 'create-jira' to push to Jira, 'fix' to address findings first, or 'skip' to stop here."

### thorough mode (progressive):

**Level 1 — Orientation:**
"Validation PASSED. Adversarial review found {n} issues. Edge case hunter found {m} unhandled paths."

**Level 2 — Walkthrough:**
Issues organized by severity (not by story order). Edge cases organized by class.
Each issue explained with its impact.

**Level 3 — Detail Pass:**
Top 5 highest-impact findings highlighted. Specific recommendations for which to fix
before Jira creation vs which can become tech debt tickets.
Tagged: `[architecture]`, `[security]`, `[scope]`, `[compliance]`

**Decision:**
"Reply 'create-jira' to push to Jira, 'fix' to address findings first, or 'skip' to stop here."

After user responds: set `checkpoint.awaiting_human` to `false`.
If user says 'skip': stop the pipeline. All local work is complete.
If user says 'fix': return to Phase 4/5 cycle with feedback.
