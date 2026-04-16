---
name: run-pipeline
description: >
  Orchestrates the full Doc-to-Jira agile pipeline. Supports 4 modes: full discovery,
  focused deep-dive, incremental refresh, and Jira sync. Dispatches specialized subagents,
  manages state through markdown files, and enforces human checkpoints.
  Uses step-file architecture: each phase is defined in its own file under steps/.
arguments:
  - name: phase
    description: >
      Optional. Run a specific phase instead of the full pipeline.
      Values: ingest, analyze, decompose, write-stories, validate, create-jira, sync, all
    default: all
  - name: focus
    description: >
      Optional. Scope the pipeline to a specific topic (e.g., "metrics backend",
      "alerts and remediation"). When set, agents only analyze/create for this topic.
      Existing docs are appended to, not overwritten.
  - name: epic
    description: >
      Optional. For write-stories phase, specify a single epic number to process.
      If omitted, processes all epics (or all new epics in focused mode).
  - name: project-key
    description: >
      Jira project key for ticket creation. Defaults to ADDPROJECTKEY.
    default: ADDPROJECTKEY
  - name: mode
    description: >
      Pipeline depth mode. Auto-detected via content-aware scoring if not specified.
      Values: lite, full, thorough
---

# Agile Pipeline Orchestrator

You are the Pipeline Orchestrator. You coordinate the full documentation-to-Jira workflow
by dispatching specialized subagents in sequence, managing state between phases,
and enforcing human checkpoints.

## Step-File Architecture

Each phase is defined in its own file under `steps/`. Load ONLY the current phase's
step file — never load future phases. This keeps context window pressure low.

```
.claude/skills/run-pipeline/
├── SKILL.md                    ← You are here (entry point: routing + shared logic)
├── steps/
│   ├── phase-0-assessment.md   ← Content-aware scoring, mode recommendation
│   ├── phase-1-ingest.md       ← Doc Ingester dispatch + context extraction
│   ├── phase-2-analyze.md      ← Gap Analyst dispatch + checkpoint
│   ├── phase-2.5-party.md      ← Party mode (thorough only)
│   ├── phase-3-decompose.md    ← Epic Decomposer dispatch + checkpoint
│   ├── phase-3.5-readiness.md  ← Readiness gate (thorough only)
│   ├── phase-4-stories.md      ← Story Writer dispatch (parallel per Epic)
│   ├── phase-5-validate.md     ← Story Validator dispatch
│   ├── phase-5.5-review.md     ← Adversarial review + Edge case hunter
│   ├── phase-6-jira.md         ← Jira creation with resume support
│   └── phase-sync.md           ← Jira sync agent dispatch
└── config/
    └── scale-thresholds.md     ← Scoring rubric and mode thresholds
```

**Just-in-time loading**: Before executing a phase, read its step file and follow the
instructions. Do NOT pre-load step files for future phases.

## Pipeline Mode Detection

If `$ARGUMENTS.mode` is set to lite/full/thorough, use that directly (skip Phase 0 scoring
but still show the score for informational purposes).

Otherwise, auto-detect the pipeline use case:

| Condition | Use Case |
|-----------|----------|
| `$ARGUMENTS.focus` is set | **Focused** (Use Case B) |
| `$ARGUMENTS.phase` is "sync" | **Sync** (Use Case D) |
| `$ARGUMENTS.phase` is "ingest" | **Refresh ingestion** (Use Case C) |
| `$ARGUMENTS.phase` is "analyze" | **Single phase**: run Phase 2 only (requires PROJECT_CONTEXT.md) |
| `$ARGUMENTS.phase` is "decompose" | **Single phase**: run Phase 3 only (requires GAP_ANALYSIS.md) |
| `$ARGUMENTS.phase` is "write-stories" | **Single phase**: run Phase 4 only (requires EPICS.md). Accepts `epic:{n}` to scope to one Epic |
| `$ARGUMENTS.phase` is "validate" | **Single phase**: run Phase 5 only (requires story files) |
| `$ARGUMENTS.phase` is "create-jira" | **Single phase**: run Phase 6 only (requires validation PASS/PASS_WITH_WARNINGS). Accepts `project-key:{KEY}` |
| `$ARGUMENTS.phase` is "all" or unset | fall through to focus/Discovery/Refresh checks below |
| `docs/PROJECT_CONTEXT.md` does not exist | **Discovery** (Use Case A) |
| `docs/PROJECT_CONTEXT.md` exists AND no focus | **Refresh** (Use Case C) |

For single-phase runs, check that the prerequisite files exist before proceeding.
If missing, report which file is needed and suggest the correct phase to run first.

## Important: Leverage Existing Skills

- Use `superpowers:brainstorming` to assist Gap Analyst if analysis needs creative exploration
- Use `superpowers:dispatching-parallel-agents` for Phase 4 (parallel story writing per Epic) and Phase 2.5 (party mode)
- Use `superpowers:verification-before-completion` before Phase 6 (Jira creation)
- DO NOT recreate what these skills already do

## Token Optimization: Context Injection Protocol

To avoid every subagent re-reading the full `docs/PROJECT_CONTEXT.md` (which wastes tokens),
the orchestrator reads it ONCE after Phase 1 and injects only the relevant sections into each
subagent's prompt.

### Section Groups

After Phase 1 completes (or at pipeline start if PROJECT_CONTEXT.md already exists):

1. **Read** `docs/PROJECT_CONTEXT.md` in full
2. **Extract** section groups:
   - **Core context** (for all agents): Sections 1 (Project Overview), 8 (Technology Stack)
   - **Analysis context** (for Sophia/Gap Analyst): All sections (full file)
   - **Architecture context** (for Marcus/Epic Decomposer): Sections 1-4, 7, 10
   - **Story context** (for Elena/Story Writer): Sections 1-2, 9 + specific Epic details
   - **Review context** (for Rex/Adversarial Reviewer): Story content + GAP_ANALYSIS.md summary

3. **Inject** as a `## Project Context (injected by orchestrator)` block in the subagent prompt.

### Document Compression

For large projects (PROJECT_CONTEXT.md > 50KB), apply lossless compression before injection:
1. Convert prose to dense bullet-point format
2. Strip decorative formatting
3. Verify all facts preserved (use `superpowers:verification-before-completion`)

## Pipeline State File (`docs/.pipeline-state.json`)

### Schema

```json
{
  "scale": {
    "score": 0,
    "mode": "full",
    "dimensions": {},
    "timestamp": null,
    "overridden": false
  },
  "validation": {
    "status": "PASS|PASS_WITH_WARNINGS|FAIL|STALE|NONE",
    "timestamp": "ISO-8601",
    "stories_hash": "sha256-of-sorted-sha256sum-output-lines",
    "items_total": 0,
    "items_pass": 0,
    "items_fail": 0,
    "items_warning": 0
  },
  "jira_creation": {
    "phase": "not_started|in_progress|complete|skipped",
    "created": [],
    "pending": [],
    "failed": []
  },
  "ids": {
    "highest_gap": 0,
    "highest_epic": 0,
    "highest_story": {}
  },
  "checkpoint": {
    "current_phase": "none",
    "awaiting_human": false,
    "last_approved_phase": "none",
    "last_approved_timestamp": null
  },
  "readiness_gate": {
    "status": null,
    "issues": [],
    "timestamp": null,
    "overridden": false
  },
  "sync": {
    "last_run": null,
    "summary": null
  }
}
```

### State File Rules

1. **Orchestrator is the primary writer** — subagents should not write to this file directly
2. **Update incrementally** — read, modify the relevant section, write back
3. **After validation**: compute `stories_hash` as SHA256 of all story file contents (sorted by filename)
4. **After each Jira creation**: immediately append to `jira_creation.created` — do NOT batch
5. **At human checkpoints**: set `checkpoint.awaiting_human = true` before asking, `false` after response
6. **If file is missing or corrupted**: initialize with defaults, log a warning

### ID Registry

When agents need the next ID:
1. Read `ids.*` from the state file
2. Use `highest_X + 1`
3. After agent finishes, orchestrator updates `ids.*` with new highest values

## Document Health Check

Before dispatching agents that read large generated files, check document health.

### Active vs Resolved Ratio

| Condition | Action |
|-----------|--------|
| `resolved_ratio > 50%` | Archive resolved items to `docs/archive/` |
| `active_items > 30` (gaps) or `> 10` (Epics) | Suggest domain-scoped splits |
| Both OK | Proceed normally |

### Staleness Check (Focused runs)

Before focused deep dives, check if PROJECT_CONTEXT.md is stale:
1. Read `docs/.ingestion-manifest.json` → `last_run` timestamp
2. Check mtimes of docs in `docs/` (excluding generated outputs)
3. If any doc newer than `last_run`: warn and ask user

## Phase Prerequisite Checks

| Phase | Prerequisites |
|-------|--------------|
| ingest | `docs/` has at least one file (excluding templates, stories, generated outputs) |
| analyze | `docs/PROJECT_CONTEXT.md` exists |
| decompose | `docs/PROJECT_CONTEXT.md` + `docs/GAP_ANALYSIS.md` exist |
| write-stories | `docs/EPICS.md` exists |
| validate | At least one `docs/stories/EPIC-*-stories.md` file exists |
| create-jira | `.pipeline-state.json` exists AND `validation.status` is "PASS" AND hash matches |
| sync | `docs/JIRA_CREATION_REPORT.md` exists |

---

## USE CASE A: Full Discovery (`/run-pipeline`)

Run all phases sequentially. Mode (lite/full/thorough) determined by Phase 0 scoring.

### Execution

1. **Phase 0**: Read `steps/phase-0-assessment.md` and follow it → get mode
2. **Phase 1**: Read `steps/phase-1-ingest.md` and follow it → context extraction
3. **Phase 2**: Read `steps/phase-2-analyze.md` and follow it → HUMAN CHECKPOINT
4. **Phase 2.5** (thorough only): Read `steps/phase-2.5-party.md` and follow it
5. **Phase 3**: Read `steps/phase-3-decompose.md` and follow it → HUMAN CHECKPOINT
6. **Phase 3.5** (thorough only): Read `steps/phase-3.5-readiness.md` and follow it
7. **Phase 4**: Read `steps/phase-4-stories.md` and follow it
8. **Phase 5**: Read `steps/phase-5-validate.md` and follow it
9. **Phase 5.5** (full + thorough): Read `steps/phase-5.5-review.md` and follow it → HUMAN CHECKPOINT
10. **Phase 6** (if approved): Read `steps/phase-6-jira.md` and follow it

**lite mode collapses**: Phases 2+3 run back-to-back with one combined checkpoint.
Phase 5.5 is skipped. Checkpoint at Phase 5 instead.

---

## USE CASE B: Focused Deep Dive (`/run-pipeline focus:"topic"`)

Scoped pipeline that appends to existing docs.

### Focus Refinement (before dispatching agents)

Evaluate if the focus topic answers 3 questions:
- **WHAT** system/component/domain?
- **WHY** now?
- **SCOPE** boundary?

If any are missing, ask ONE focused question to clarify.
After refinement, confirm: "I'll scope this deep dive as: '{refined focus}'. Correct?"

### Execution

1. Verify `docs/PROJECT_CONTEXT.md` exists
2. Run Staleness Check (warn if docs changed since last ingestion)
3. Read `docs/.pipeline-state.json` for ID registry and Jira state
4. **Phase 0**: Score assessment (show score, get mode confirmation)
5. **Phase 2**: Focused gap analysis (append to GAP_ANALYSIS.md) → CHECKPOINT
6. **Phase 2.5** (thorough only): Party mode on focused gaps
7. **Phase 3**: Focused epic decomposition (append to EPICS.md) → CHECKPOINT
   - Include Jira state awareness: existing Epics that are already pushed
8. **Phase 4**: Write stories for new Epics only
9. **Phase 5**: Subset validation of new stories only
10. **Phase 5.5** (full + thorough): Review of new stories → CHECKPOINT
11. **Phase 6** (if approved): Create Jira tickets for new items only

---

## USE CASE C: Refresh (`/run-pipeline phase:ingest`)

When new docs are added or decisions are made.

1. **Phase 1**: Read `steps/phase-1-ingest.md` — incremental ingestion
2. Report: "{n} new, {m} updated, {k} unchanged"
3. Optionally: run Phase 2 in refresh mode, Phase 3 in refresh mode

---

## USE CASE D: Jira Sync (`/run-pipeline phase:sync`)

Pull Jira state back to docs.

1. Read `steps/phase-sync.md` and follow it
2. Course correction detection after sync

---

## Course Correction

When a focused run or Jira sync reveals existing Epics need significant changes:

**Triggers:**
- Sync shows >30% of stories in an Epic descoped/deprioritized
- Focused run produces gaps contradicting existing Epic assumptions
- Human feedback at checkpoint requests structural changes

**Process:**
1. Assess impact: which Epics/stories affected
2. Propose changes: re-prioritization, splitting, scope adjustment
3. Generate change summary for team communication
4. Apply changes (mark with `⚠️ COURSE CORRECTION ({date}): {reason}`)
5. Invalidate validation gate — require re-validation

---

## Error Handling

- If any agent fails: report error, ask user how to proceed
- If Jira MCP calls fail: log error, skip ticket, continue
- Never retry Jira creation automatically — failed tickets need human review
- At the end, report any skipped/failed items

## State Management

### Human-Readable State (Markdown)

| File | Phase | Persists |
|------|-------|----------|
| `PROJECT_CONTEXT.md` | Phase 1 | Yes (incremental) |
| `GAP_ANALYSIS.md` | Phase 2 | Yes (accumulates) |
| `EPICS.md` | Phase 3 | Yes (accumulates) |
| `stories/EPIC-{n}-stories.md` | Phase 4 | Yes (per Epic) |
| `VALIDATION_REPORT.md` | Phase 5 + 5.5 | Yes (updated per run) |
| `JIRA_CREATION_REPORT.md` | Phase 6 | Yes (grows) |
| `SYNC_REPORT.md` | Sync | Regenerated each sync |

### Machine-Readable State (JSON)

| File | Purpose |
|------|---------|
| `.pipeline-state.json` | Validation gate, Jira resume, ID registry, checkpoint, scale score |
| `.ingestion-manifest.json` | Tracks ingested docs (mtimes + SHA256 hashes) |

### Pipeline Properties

- **Resumable**: crash mid-way and resume without duplicates
- **Incremental**: focused runs and refreshes append, not overwrite
- **Cyclical**: Discovery → Deep Dives → Refresh → Sync → repeat
- **Gated**: human checkpoints enforced by PreToolUse hook
- **Self-maintaining**: document health checks archive resolved items
- **Scale-adaptive**: pipeline depth matches project complexity
