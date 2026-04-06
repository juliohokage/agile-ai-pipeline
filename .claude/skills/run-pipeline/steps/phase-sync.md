# Phase: Jira Sync

## When This Runs
On demand — `/run-pipeline phase:sync`.

## Agent
**Dash** (Jira Sync) — `.claude/agents/jira-sync.md`

## Input
- `docs/JIRA_CREATION_REPORT.md` — local-to-Jira mapping
- Jira MCP queries for current ticket states
- `docs/EPICS.md`, `docs/stories/` — for comparison

## Process

### Step 1: Verify prerequisites

- Confirm `docs/JIRA_CREATION_REPORT.md` exists
- If not: "No Jira creation report found. Run `/run-pipeline phase:create-jira` first,
  or if tickets were created manually, create a mapping file."

### Step 2: Dispatch Jira Sync agent

Launch the `jira-sync` subagent via the Agent tool:

"You are Dash. Read docs/JIRA_CREATION_REPORT.md for the local-to-Jira mapping.
Query Jira using jira_search for all issues in project {project-key}.
Compare against docs/EPICS.md and docs/stories/.
Produce docs/SYNC_REPORT.md."

### Step 3: Verify output

- Confirm `docs/SYNC_REPORT.md` was created
- Extract summary: progress percentages, status changes, new issues, missing issues

### Step 4: Present results

Set `checkpoint.awaiting_human` to `true`.

"Sync complete. {summary}.
Review `docs/SYNC_REPORT.md`.
Would you like me to update the local docs to reflect Jira state?
Reply 'update-docs' to proceed, or 'skip' to keep docs as-is."

### Step 5: Update docs (if approved)

If user says 'update-docs':
- Re-launch `jira-sync` with prompt: "Update the local docs based on the approved
  SYNC_REPORT.md. Add status badges to stories, update Epic progress in EPICS.md,
  mark resolved gaps in GAP_ANALYSIS.md."

### Step 6: Course correction detection

After sync, check for course correction triggers:
- If >30% of stories in any Epic were descoped or deprioritized in Jira
- If new Jira issues exist that don't map to local stories
- If significant status drift is detected

If triggers found, suggest: "Course correction may be needed. {reason}.
Run `/run-pipeline focus:'{topic}'` to investigate, or review the sync report first."

## Output
- `docs/SYNC_REPORT.md` — diff between Jira state and local docs

## State Updates
```json
{
  "checkpoint": {
    "current_phase": "sync",
    "awaiting_human": true
  },
  "sync": {
    "last_run": "{ISO-8601}",
    "summary": "{brief summary}"
  }
}
```

## Checkpoint
Human checkpoint after sync report is presented — user decides whether to update local docs.
After user responds: set `checkpoint.awaiting_human` to `false`.
