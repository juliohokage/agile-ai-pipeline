# Phase 6: Jira Creation

## When This Runs
Only if user explicitly requests it (says 'create-jira' at the Phase 5/5.5 checkpoint).

## Agent
Inline (orchestrator makes direct Jira MCP calls, no subagent dispatch).

## Input
- `docs/stories/EPIC-*-stories.md` — validated story files
- `docs/EPICS.md` — Epic definitions
- `docs/.pipeline-state.json` — validation gate, resume state, ID registry
- `$ARGUMENTS.project-key` — Jira project key

## Process

### Step 1: Pre-flight checks

Use `superpowers:verification-before-completion` to verify:
1. `docs/.pipeline-state.json` → `validation.status` == "PASS"
2. Re-compute stories hash, compare against `validation.stories_hash`
3. User has explicitly approved Jira creation (at the Phase 5/5.5 checkpoint)
4. Project key is configured (not "ADDPROJECTKEY" unless user confirmed it)

If any check fails, stop and report.

### Step 2: Resume check

Read `docs/.pipeline-state.json` → `jira_creation`:
- If `phase` is "in_progress" and `created` is non-empty → **resumed run**
- Skip items already in `created`, continue from `pending`
- Report: "Resuming Jira creation. {n} items already created, {m} remaining."

### Step 3: Initialize state

If not resuming:
- Set `jira_creation.phase` to "in_progress"
- Populate `jira_creation.pending` with all local IDs (Epics + Stories + Spikes)

### Step 4: Create Epics

For each Epic in `docs/EPICS.md` (skip if already in `created`):
```
Call: mcp__atlassian__jira_create_issue
  project_key: {project-key}
  issue_type: "Epic"
  summary: "{Epic Name}"
  description: "{Epic description, scope, success criteria}"
```
**Immediately** after creation:
- Record the Jira key (e.g., PROJ-1)
- Append to `jira_creation.created`
- Remove from `jira_creation.pending`
- Write updated state to `docs/.pipeline-state.json`

### Step 5: Create Stories and Spikes

For each item in each `docs/stories/EPIC-{n}-stories.md` (skip if already in `created`):
```
Call: mcp__atlassian__jira_create_issue
  project_key: {project-key}
  issue_type: "Story"
  summary: "{title}" (prefix Spikes with "[SPIKE] ")
  description: |
    **As a** {role}
    **I want** {goal}
    **So that** {benefit}

    **Type**: {Story | Spike}
    **Time-box**: {for Spikes only}

    ## Acceptance Criteria
    {Gherkin scenarios}

    ## Technical Notes
    {notes}

    ## Open Questions
    {questions}

    ## Output
    {for Spikes: expected deliverable}
  priority: {Must Have=Highest, Should Have=High, Could Have=Medium}
```
**After each creation**: immediately update state (append to `created`, remove from `pending`).
If creation fails: add to `jira_creation.failed` with error, continue with remaining items.

### Step 6: Link items to Epics

```
Call: mcp__atlassian__jira_link_to_epic
  issue_key: {item key}
  epic_key: {parent epic key}
```

### Step 7: Create dependency links

For each declared dependency between Epics/Stories:
```
Call: mcp__atlassian__jira_create_issue_link
  link_type: "Blocks"
  inward_issue: {blocking item key}
  outward_issue: {blocked item key}
```

### Step 8: Save creation report

Write `docs/JIRA_CREATION_REPORT.md`:
```markdown
# Jira Creation Report

Generated: {date}
Project: {PROJECT_KEY}

## Mapping

| Local ID | Jira Key | Type | Summary | Epic Key |
|----------|----------|------|---------|----------|
| EPIC-1 | PROJ-1 | Epic | {name} | — |
| US-1.1 | PROJ-2 | Story | {name} | PROJ-1 |

## Summary
Total: {n} Epics, {m} Stories, {k} Spikes created

## Failed (if any)
| Local ID | Error | Action Needed |
|----------|-------|---------------|
```

### Step 9: Finalize state

Set `jira_creation.phase` to "complete" in `docs/.pipeline-state.json`.

## Output
- `docs/JIRA_CREATION_REPORT.md` — mapping of local IDs to Jira keys
- Jira tickets created

## State Updates
```json
{
  "jira_creation": {
    "phase": "complete",
    "created": [{"local_id": "...", "jira_key": "...", "type": "..."}],
    "pending": [],
    "failed": []
  }
}
```

## Checkpoint
None — the human checkpoint was at Phase 5/5.5. Jira creation proceeds automatically
once approved. Report results when complete.
