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

### Step 5: Intelligent next-action recommendations

Score each potential action by **urgency** (how time-sensitive) and **impact** (how much it
unblocks). Present the top 3-5 actions ranked by combined score.

#### Scoring signals to check:

**Pipeline not started** (urgency: HIGH, impact: HIGH):
- If no `docs/PROJECT_CONTEXT.md` exists → "Run `/run-pipeline` to start discovery"

**Critical gaps unresolved** (urgency: HIGH, impact: HIGH):
- If GAP_ANALYSIS.md has critical/high open gaps → "Review GAP_ANALYSIS.md — {n} critical gaps need stakeholder decisions"

**Validation stale or failed** (urgency: HIGH, impact: MEDIUM):
- Read `docs/.pipeline-state.json` → if `validation.status` is "STALE" or "FAIL"
  → "Re-validate stories — validation is {STALE|FAILED}. Run `/run-pipeline phase:validate`"

**Ingestion stale** (urgency: MEDIUM, impact: MEDIUM):
- Read `docs/.ingestion-manifest.json` → compare `last_run` against doc mtimes
- If any doc is newer than last ingestion → "Re-ingest docs — {n} files changed since last ingestion on {date}. Run `/run-pipeline phase:ingest`"

**Jira sync stale** (urgency: MEDIUM, impact: LOW):
- If `docs/JIRA_CREATION_REPORT.md` exists but `docs/SYNC_REPORT.md` doesn't exist or is old
  → "Sync with Jira — tickets exist but haven't been synced{' since {date}' if sync report exists}. Run `/run-pipeline phase:sync`"

**Epics without stories** (urgency: LOW, impact: MEDIUM):
- Cross-reference EPICS.md with story files → if any Epic has no story file
  → "Write stories for EPIC-{n} — no stories exist yet. Run `/run-pipeline phase:write-stories epic:{n}`"

**Adversarial review not run** (urgency: LOW, impact: LOW):
- If VALIDATION_REPORT.md exists but has no "Adversarial Review Findings" section
  → "Consider running adversarial review for stronger validation"

**Scale assessment** (informational):
- If `docs/.pipeline-state.json` has `scale` data → show: "Pipeline mode: {mode} (score: {score}/30)"

```markdown
### Next Actions (ranked by urgency × impact)

1. **[URGENT]** {action} — {reason}
2. **[RECOMMENDED]** {action} — {reason}
3. **[OPTIONAL]** {action} — {reason}
```

### Step 6: Mode recommendation (if pipeline hasn't been run)

If no pipeline outputs exist at all:
- Count docs in `docs/` (excluding generated files)
- Suggest: "You have {n} documentation files. Run `/run-pipeline` to start.
  The pipeline will assess project complexity and recommend a mode (lite/full/thorough)."

## Rules

- Read-only — never modify any files
- Use last available data — don't query Jira (use `/sprint-health` for live Jira data)
- If no pipeline outputs exist, say so and suggest running `/run-pipeline`
- Keep the output concise — this is a quick status check, not a deep analysis
- Always end with actionable next steps ranked by urgency and impact
- Check `docs/.pipeline-state.json` for validation state and scale score when available
