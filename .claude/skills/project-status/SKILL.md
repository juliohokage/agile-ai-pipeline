---
name: project-status
description: >
  Quick project status overview from local docs and optionally Jira.
  Shows Epic progress, open gaps, recent sync data, and what needs attention.
  Use when someone asks "where are we?" without running a full sync.
---

# Project Status Overview

Quick status check from local documentation. No Jira queries required
(uses last sync data if available).

## Process

### Step 1: Check what data is available

Look for these files and note which exist:
- `docs/PROJECT_CONTEXT.md` — project understood?
- `docs/GAP_ANALYSIS.md` — gaps identified?
- `docs/EPICS.md` — epics defined?
- `docs/stories/EPIC-*-stories.md` — stories written?
- `docs/VALIDATION_REPORT.md` — stories validated?
- `docs/JIRA_CREATION_REPORT.md` — pushed to Jira?
- `docs/SYNC_REPORT.md` — synced with Jira?

### Step 2: Build status report

```markdown
## Project Status

**Date**: {date}
**Pipeline State**: {where the project is in the pipeline lifecycle}

### Pipeline Progress

| Phase | Status | Output |
|-------|--------|--------|
| 1. Doc Ingestion | {Done / Not run} | {n} docs ingested |
| 2. Gap Analysis | {Done / Not run} | {n} gaps ({open} open, {resolved} resolved) |
| 3. Epic Decomposition | {Done / Not run} | {n} Epics defined |
| 4. Story Writing | {Done / Partial / Not run} | {n} Stories, {k} Spikes across {e} Epics |
| 5. Validation | {Passed / Failed / Not run} | {pass}/{total} items passing |
| 6. Jira Creation | {Done / Not run} | {n} tickets created |
| 7. Last Sync | {date or Never} | {summary if available} |
```

### Step 3: Epic status (if EPICS.md exists)

Read `docs/EPICS.md` and present the overview table.
If `docs/SYNC_REPORT.md` exists, overlay progress percentages from last sync.

```markdown
### Epic Overview

| Epic | Priority | Stories | Progress | Status |
|------|----------|---------|----------|--------|
| EPIC-1: {name} | Must Have | 5 | 60% (from last sync) | Active |
| EPIC-2: {name} | Must Have | 4 | 0% | Not started |
```

### Step 4: Open gaps summary (if GAP_ANALYSIS.md exists)

Read `docs/GAP_ANALYSIS.md` and show only OPEN gaps (skip resolved):

```markdown
### Open Gaps ({n} remaining)

**Critical**:
- {gap ID}: {description}

**High Priority**:
- {gap ID}: {description}

**Unanswered Questions**: {n}
```

### Step 5: What needs attention

Based on all available data, list what should happen next:

```markdown
### Next Actions

1. {Most urgent action based on current state}
2. {Second priority}
3. ...
```

Examples:
- "Run `/run-pipeline` — pipeline hasn't been started yet"
- "Run `/run-pipeline phase:sync` — last sync was 5 days ago"
- "Review GAP_ANALYSIS.md — 3 critical gaps need stakeholder decisions"
- "Fix validation failures — 2 stories failed INVEST checks"
- "Run `/run-pipeline focus:'alerts'` — EPIC-3 has no stories yet"

## Rules

- Read-only — never modify any files
- Use last available data — don't query Jira (use `/sprint-health` for live Jira data)
- If no pipeline outputs exist, say so and suggest running `/run-pipeline`
- Keep the output concise — this is a quick status check, not a deep analysis
- Always end with actionable next steps
