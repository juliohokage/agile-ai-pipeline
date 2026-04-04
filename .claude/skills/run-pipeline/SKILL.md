---
name: run-pipeline
description: >
  Orchestrates the full Doc-to-Jira agile pipeline. Supports 4 modes: full discovery,
  focused deep-dive, incremental refresh, and Jira sync. Dispatches specialized subagents,
  manages state through markdown files, and enforces human checkpoints.
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
      Pipeline execution mode. Auto-detected if not specified.
      Values: discovery, focused, refresh, sync
---

# Agile Pipeline Orchestrator

You are the Pipeline Orchestrator. You coordinate the full documentation-to-Jira workflow
by dispatching specialized subagents in sequence, managing state between phases,
and enforcing human checkpoints.

## Mode Detection

If `$ARGUMENTS.mode` is not specified, auto-detect:

| Condition | Mode |
|-----------|------|
| `$ARGUMENTS.focus` is set | **Focused** |
| `$ARGUMENTS.phase` is "sync" | **Sync** |
| `docs/PROJECT_CONTEXT.md` does not exist | **Discovery** (first run) |
| `docs/PROJECT_CONTEXT.md` exists AND no focus | **Refresh** |

## Use Cases Overview

```
Use Case A: Discovery   → /run-pipeline                    (first run, full analysis)
Use Case B: Deep Dive   → /run-pipeline focus:"topic"      (scoped to a topic, appends)
Use Case C: Refresh     → /run-pipeline phase:ingest       (re-ingest changed docs, update)
Use Case D: Sync        → /run-pipeline phase:sync         (pull Jira state back to docs)
```

## Important: Leverage Existing Skills

- Use `superpowers:brainstorming` to assist Gap Analyst if analysis needs creative exploration
- Use `superpowers:dispatching-parallel-agents` for Phase 4 (parallel story writing per Epic)
- Use `superpowers:verification-before-completion` before Phase 6 (Jira creation)
- DO NOT recreate what these skills already do

## Token Optimization: Context Injection

To avoid every subagent re-reading the full `docs/PROJECT_CONTEXT.md` (which wastes tokens),
the orchestrator reads it ONCE after Phase 1 and injects only the relevant sections into each
subagent's prompt.

### Context Injection Protocol

After Phase 1 completes (or at pipeline start if PROJECT_CONTEXT.md already exists):

1. **Read** `docs/PROJECT_CONTEXT.md` in full
2. **Extract** the following section groups:
   - **Core context** (for all agents): Sections 1 (Project Overview), 8 (Technology Stack)
   - **Analysis context** (for Gap Analyst): All sections (full file — this agent needs everything)
   - **Architecture context** (for Epic Decomposer): Sections 1-4 (Overview, Architecture, Stakeholders, Decisions) + 7 (Constraints) + 10 (Open Items)
   - **Story context** (for Story Writer): Sections 1-2 (Overview, Architecture) + 9 (Entities & Domain Model) — plus the specific Epic details from EPICS.md
   - **Validation context** (for Story Validator): Not needed — validator reads story files only

3. **Inject** the extracted text directly into the subagent prompt as a `## Project Context (injected)` block,
   so the subagent does NOT need to re-read the file.

### Injection format in subagent prompts

```
## Project Context (injected by orchestrator — do NOT re-read PROJECT_CONTEXT.md)

{extracted sections here}
```

When injecting context, include a note about which sections were included and which were omitted,
so the subagent knows its context boundaries. If the subagent needs a section not provided,
it should read PROJECT_CONTEXT.md itself (fallback).

## Pipeline State File (`docs/.pipeline-state.json`)

The pipeline maintains a machine-readable state file for integrity enforcement.
This file is the backbone for hooks, resumability, and ID management.

### Schema

```json
{
  "validation": {
    "status": "PASS|FAIL|STALE|NONE",
    "timestamp": "ISO-8601",
    "stories_hash": "sha256-of-all-story-files-combined"
  },
  "jira_creation": {
    "phase": "not_started|in_progress|complete",
    "created": [
      {"local_id": "EPIC-1", "jira_key": "PROJ-1", "type": "Epic"},
      {"local_id": "US-1.1", "jira_key": "PROJ-2", "type": "Story"}
    ],
    "pending": ["US-1.3", "US-1.4"],
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
  }
}
```

### State File Rules

1. **Orchestrator is the primary writer** — subagents should not write to this file directly
2. **Update incrementally** — read, modify the relevant section, write back
3. **After validation**: compute `stories_hash` as SHA256 of all story file contents (sorted by filename), store in `validation.stories_hash`
4. **After each Jira creation**: immediately append to `jira_creation.created` and remove from `pending` — do NOT wait until the end
5. **At human checkpoints**: set `checkpoint.awaiting_human = true` before asking, set to `false` after user responds
6. **If file is missing or corrupted**: initialize a fresh state file with all defaults, log a warning

### ID Registry

When agents need the next ID (gap, epic, story), they should:
1. Read `ids.*` from the state file
2. Use `highest_X + 1`
3. After the agent finishes, the orchestrator updates `ids.*` with the new highest values

This prevents ID collisions across focused runs. The orchestrator passes the next available ID range to each subagent in its prompt.

## Document Health Check

Before dispatching agents that read large generated files, the orchestrator checks document health.

### Active vs Resolved Ratio

For files that accumulate items across runs (GAP_ANALYSIS.md, EPICS.md, story files):

1. **Count active items**: Open gaps, active Epics, unfinished stories
2. **Count resolved items**: Resolved gaps, completed Epics, done stories
3. **Calculate ratio**: `resolved / total`

| Condition | Action |
|-----------|--------|
| `resolved_ratio > 50%` | Archive resolved items to `docs/archive/` — move detailed entries, keep one-line summary in main file |
| `active_items > 30` (gaps) or `> 10` (Epics) | Suggest domain-scoped splits: "This file has {n} active items. Consider running focused deep dives per domain instead." |
| Both factors OK | Proceed normally regardless of line count |

### Archival Process

When archival is triggered:
1. Create `docs/archive/{filename}-archive-{date}.md`
2. Move resolved/completed item details to the archive file
3. In the main file, replace each archived item with a one-line summary: `| RG-5 | ... | ✅ Resolved | Archived: docs/archive/... |`
4. Update the state file with new item counts

### Staleness Check (Use Case B)

Before starting a focused deep dive, check if PROJECT_CONTEXT.md is stale:
1. Read `docs/.ingestion-manifest.json` → get `last_run` timestamp
2. Check mtimes of all docs in `docs/` (excluding generated outputs, templates, stories)
3. If any doc has mtime newer than `last_run`:
   - Warn: "PROJECT_CONTEXT.md may be stale. {n} documents changed since last ingestion on {date}."
   - Ask: "Run `/run-pipeline phase:ingest` first, or proceed with current context?"
   - Wait for user response

## Phase Prerequisite Checks

Before running any phase, verify prerequisites:

| Phase | Prerequisites |
|-------|--------------|
| ingest | `docs/` directory has at least one file (excluding templates, stories, generated outputs) |
| analyze | `docs/PROJECT_CONTEXT.md` exists |
| decompose | `docs/PROJECT_CONTEXT.md` + `docs/GAP_ANALYSIS.md` exist |
| write-stories | `docs/EPICS.md` exists |
| validate | At least one `docs/stories/EPIC-*-stories.md` file exists |
| create-jira | `docs/.pipeline-state.json` exists AND `validation.status` is "PASS" AND `validation.stories_hash` matches current story files |
| sync | `docs/JIRA_CREATION_REPORT.md` exists |

If prerequisites are missing, tell the user which phase needs to run first.

---

## USE CASE A: Full Discovery (`/run-pipeline`)

Run all phases sequentially. This is the first-run full analysis.

### Phase 1: Documentation Ingestion

```
Dispatch: doc-ingester agent (full mode)
Input: docs/ directory
Output: docs/PROJECT_CONTEXT.md + docs/.ingestion-manifest.json
```

1. Verify `docs/` contains at least one documentation file
2. Launch the `doc-ingester` subagent via the Agent tool with prompt:
   "Run in FULL INGESTION mode. Read all documents in docs/ and create docs/PROJECT_CONTEXT.md and docs/.ingestion-manifest.json."
3. Verify `docs/PROJECT_CONTEXT.md` was created
4. **Read `docs/PROJECT_CONTEXT.md` now** and store its content for context injection into subsequent agents (see "Token Optimization: Context Injection" section above)
5. Report: "Phase 1 complete. PROJECT_CONTEXT.md generated from {n} documents."

### Phase 2: Gap Analysis

```
Dispatch: gap-analyst agent (full mode)
Input: docs/PROJECT_CONTEXT.md
Output: docs/GAP_ANALYSIS.md
```

1. Launch the `gap-analyst` subagent via the Agent tool with prompt that includes:
   - "Run in FULL ANALYSIS mode. Create docs/GAP_ANALYSIS.md."
   - **Inject: full PROJECT_CONTEXT.md content** (analysis context — all sections needed)
2. Verify `docs/GAP_ANALYSIS.md` was created
3. Report summary: gap counts by type, critical risks

3. After gap-analyst completes, update `docs/.pipeline-state.json`:
   - Update `ids.highest_gap` with the highest gap ID produced
   - Set `checkpoint.current_phase` to "analyze"
   - Set `checkpoint.awaiting_human` to `true`

**HUMAN CHECKPOINT**: Stop and present the gap analysis summary.
Ask: "Gap analysis complete. {n} gaps identified ({critical} critical). Please review `docs/GAP_ANALYSIS.md`. Reply 'proceed' to continue to Epic decomposition, or provide feedback."

Wait for user response. Do NOT proceed until they confirm.
After user confirms: set `checkpoint.awaiting_human` to `false`, set `checkpoint.last_approved_phase` to "analyze".

### Phase 3: Epic Decomposition

```
Dispatch: epic-decomposer agent (full mode)
Input: docs/PROJECT_CONTEXT.md + docs/GAP_ANALYSIS.md
Output: docs/EPICS.md
```

1. Launch the `epic-decomposer` subagent via the Agent tool with prompt that includes:
   - "Run in FULL DECOMPOSITION mode. Read docs/GAP_ANALYSIS.md. Create docs/EPICS.md. Follow the template in docs/templates/epic-template.md."
   - **Inject: architecture context** (sections 1-4, 7, 10 from PROJECT_CONTEXT.md)
2. Verify `docs/EPICS.md` was created
3. Report: Epic overview table and dependency graph

3. After epic-decomposer completes, update `docs/.pipeline-state.json`:
   - Update `ids.highest_epic` with the highest Epic number produced
   - Set `checkpoint.current_phase` to "decompose"
   - Set `checkpoint.awaiting_human` to `true`

**HUMAN CHECKPOINT**: Stop and present the Epic structure.
Ask: "Epic decomposition complete. {n} Epics identified. Please review `docs/EPICS.md`. Reply 'proceed' to continue to Story writing, or provide adjustments."

Wait for user response. Do NOT proceed until they confirm.
After user confirms: set `checkpoint.awaiting_human` to `false`, set `checkpoint.last_approved_phase` to "decompose".

### Phase 4: Story Writing (Parallelizable)

```
Dispatch: story-writer agent (one per Epic)
Input: docs/PROJECT_CONTEXT.md + one Epic from docs/EPICS.md
Output: docs/stories/EPIC-{n}-stories.md (one file per Epic)
```

1. Parse `docs/EPICS.md` to get the list of Epics
2. If `$ARGUMENTS.epic` is specified, only process that one Epic
3. Otherwise, use `superpowers:dispatching-parallel-agents` to launch one story-writer
   subagent per Epic in parallel. Each subagent receives:
   - **Inject: story context** (sections 1-2, 9 from PROJECT_CONTEXT.md) — do NOT tell the agent to re-read the full file
   - Its assigned Epic number and full Epic details extracted from EPICS.md
   - Instructions to read docs/templates/story-template.md for format
   - Instructions to output to `docs/stories/EPIC-{n}-stories.md`
4. Verify all story files were created
5. Report: Total items (stories + spikes) generated per Epic

### Phase 5: Story Validation

```
Dispatch: story-validator agent (full mode)
Input: All docs/stories/EPIC-*-stories.md files
Output: docs/VALIDATION_REPORT.md
```

1. Launch the `story-validator` subagent via the Agent tool with prompt:
   "Run in FULL VALIDATION mode. Read all docs/stories/EPIC-*-stories.md files. Create docs/VALIDATION_REPORT.md."
2. Verify `docs/VALIDATION_REPORT.md` was created
3. Check if "Ready for Jira: YES" is in the report
4. **Update `docs/.pipeline-state.json`**:
   - If PASS: set `validation.status` to "PASS"
   - If FAIL: set `validation.status` to "FAIL"
   - Set `validation.timestamp` to current ISO-8601 timestamp
   - Compute `validation.stories_hash`: run `find docs/stories -name "EPIC-*-stories.md" -exec sha256sum {} \; | sort | sha256sum` and store the result
   - Update `ids.highest_story` with the highest story number per Epic

If validation FAILS:
- Report the failures to the user
- Ask: "Validation found {n} issues. Would you like me to re-run the story-writer for the affected Epics with the feedback, or do you want to fix them manually?"
- If auto-fix: re-run story-writer for affected Epics, then re-validate

**HUMAN CHECKPOINT**:
- Set `checkpoint.current_phase` to "validate", `checkpoint.awaiting_human` to `true`
- Stop and present validation results.
- Ask: "Validation {PASSED|FAILED}. {summary}. Review `docs/VALIDATION_REPORT.md`. Reply 'create-jira' to push to Jira, 'skip' to stop here, or provide feedback."
- Wait for user response. After user confirms: set `checkpoint.awaiting_human` to `false`.
- **If user says 'skip' or declines Jira creation**: stop the pipeline here. All local work is complete.

### Phase 6: Jira Creation

```
Tool: Jira MCP (mcp-atlassian)
Input: Validated stories from docs/stories/
Output: Jira tickets + docs/JIRA_CREATION_REPORT.md
```

**Pre-flight checks** (use `superpowers:verification-before-completion`):
1. Confirm `docs/.pipeline-state.json` has `validation.status` = "PASS" and stories hash is current
2. Confirm the user has explicitly approved Jira creation
3. Confirm project key is set (not "ADDPROJECTKEY" unless user confirmed it)

**Resume check**: Read `docs/.pipeline-state.json` → `jira_creation`. If `phase` is "in_progress" and `created` is non-empty, this is a **resumed run**. Skip items already in `created`, continue from `pending`.

**Execution**:

Before starting, set `jira_creation.phase` to "in_progress" and populate `jira_creation.pending` with all local IDs to create.

1. **Create Epics first**:
   For each Epic in `docs/EPICS.md`:
   ```
   Call: jira_create_issue
     project_key: $ARGUMENTS.project-key
     issue_type: "Epic"
     summary: "{Epic Name}"
     description: "{Epic description, scope, success criteria}"
   ```
   Record the created Epic issue key (e.g., PROJ-1)
   **Immediately** update `docs/.pipeline-state.json`: append to `jira_creation.created`, remove from `pending`

2. **Create Stories and Spikes under Epics**:
   For each item in each `docs/stories/EPIC-{n}-stories.md`:
   ```
   Call: jira_create_issue
     project_key: $ARGUMENTS.project-key
     issue_type: "Story"  (use "Story" for both Stories and Spikes)
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
   **After each story creation**: immediately update `docs/.pipeline-state.json` — append to `created`, remove from `pending`
   If a story creation fails: add to `jira_creation.failed` with the error, continue with remaining items

3. **Link items to Epics**:
   ```
   Call: jira_link_to_epic
     issue_key: {item key}
     epic_key: {parent epic key}
   ```

4. **Create dependency links**:
   ```
   Call: jira_create_issue_link
     link_type: "Blocks"
     inward_issue: {blocking item key}
     outward_issue: {blocked item key}
   ```

5. **Save creation report**:
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
   | SPIKE-1.2 | PROJ-3 | Spike | {name} | PROJ-1 |

   ## Summary
   Total: {n} Epics, {m} Stories, {k} Spikes created

   ## Failed (if any)
   | Local ID | Error | Action Needed |
   |----------|-------|---------------|
   ```

6. **Finalize state**: Set `jira_creation.phase` to "complete" in `docs/.pipeline-state.json`

---

## USE CASE B: Focused Deep Dive (`/run-pipeline focus:"topic"`)

Scoped pipeline that appends to existing docs.

### Phase 0: Focus Refinement

Before dispatching any agent, evaluate the focus topic clarity.

1. **Check if the focus topic answers these 3 questions**:
   - **WHAT** system/component/domain? (e.g., "metrics backend" vs just "metrics")
   - **WHY** now? (e.g., "need to choose between Mimir and VictoriaMetrics")
   - **SCOPE** boundary? (e.g., "only ingestion pipeline, not dashboards")

2. **If all 3 are clear** → proceed directly.
   Example: `focus:"metrics backend — evaluate Mimir vs VictoriaMetrics for ingestion only"`
   → WHAT: metrics backend, WHY: tech choice needed, SCOPE: ingestion only

3. **If any are missing** → ask the user ONE focused question:
   - Missing WHAT: "Your topic '{focus}' could cover several areas. Which specific component: {list options from PROJECT_CONTEXT.md}?"
   - Missing WHY: "What's driving this deep dive? (a) tech decision needed, (b) gap found during sprint, (c) stakeholder request, (d) other?"
   - Missing SCOPE: "Should this cover the full {topic} domain, or a specific slice? What's explicitly OUT of scope?"

4. **After refinement**, compose the enhanced focus statement and confirm:
   "I'll scope this deep dive as: '{refined focus}'. Correct?"

5. Pass the refined focus to Phase 2.

**Note**: If the topic reveals a genuinely large/unclear area that needs decomposition, suggest: "This topic is too broad for a single focused run. Consider running `superpowers:brainstorming` to scope it into specific sub-topics first, then run separate focused pipelines for each."

### Staleness Check

Before proceeding:
1. Read `docs/.ingestion-manifest.json` → get `last_run` timestamp
2. Check mtimes of docs in `docs/` (excluding generated outputs, templates, stories)
3. If any doc has mtime newer than `last_run`:
   - Warn: "PROJECT_CONTEXT.md may be stale. {n} documents changed since last ingestion on {date}."
   - Ask: "Run `/run-pipeline phase:ingest` first, or proceed with current context?"
   - Wait for user response

### Behavior Changes

| Phase | Change vs Full Discovery |
|-------|------------------------|
| Phase 1 (Ingest) | **SKIPPED** — context already loaded |
| Phase 2 (Analyze) | Focused mode — only gaps related to the focus topic. Append to existing GAP_ANALYSIS.md |
| Phase 3 (Decompose) | Focused mode — create/extend Epics for the focus topic only. Append to EPICS.md |
| Phase 4 (Write) | Write stories for new/extended Epics only |
| Phase 5 (Validate) | Subset mode — validate only the new stories |
| Phase 6 (Jira) | Create only the new tickets |

### Cross-Run Dependency Awareness (H5)

Before Phase 3, if `docs/.pipeline-state.json` exists and `jira_creation.created` is non-empty,
or if `docs/SYNC_REPORT.md` exists:
- Read the Jira creation state to know which Epics have been pushed and their current status
- Inject this context into the Epic Decomposer's prompt:
  "The following Epics already exist in Jira: {list with status}. Do not create work that duplicates completed Epics. New Epics may depend on existing ones — reference them by number."

### Execution

1. Verify `docs/PROJECT_CONTEXT.md` exists (if not, tell user to run full discovery first)
2. Run **Phase 0** (Focus Refinement) — refine the focus topic if needed
3. Run **Staleness Check** — warn if docs changed since last ingestion
4. Read `docs/.pipeline-state.json` for ID registry and Jira state (if exists)

5. **Phase 2 — Focused Gap Analysis**:
   Launch `gap-analyst` with prompt that includes:
   - "Run in FOCUSED ANALYSIS mode. Focus topic: '{refined focus}'. Read existing docs/GAP_ANALYSIS.md. Only analyze gaps related to '{refined focus}'. Append new gaps starting from ID {next_gap_id}."
   - **Inject: full PROJECT_CONTEXT.md content**
   - After completion: update `ids.highest_gap` in state file

   **HUMAN CHECKPOINT**: Set `checkpoint.awaiting_human = true`. Present focused gaps. Wait for approval. Then set `awaiting_human = false`.

6. **Phase 3 — Focused Epic Decomposition**:
   Launch `epic-decomposer` with prompt that includes:
   - "Run in FOCUSED DECOMPOSITION mode. Focus topic: '{refined focus}'. Read docs/GAP_ANALYSIS.md and existing docs/EPICS.md. Create or extend Epics ONLY for '{refined focus}'. Start numbering from EPIC-{next_epic_number}."
   - **Inject: architecture context** (sections 1-4, 7, 10) + Jira state awareness (see above)
   - After completion: update `ids.highest_epic` in state file

   **HUMAN CHECKPOINT**: Set `checkpoint.awaiting_human = true`. Present new/updated Epics. Wait for approval.

7. **Phase 4 — Write stories** for new Epics only (with story context injection + ID ranges from state file)
8. **Phase 5 — Validate** new stories only (subset mode), update validation state
9. **Phase 6 — Create Jira tickets** for new items only (with resume support)

---

## USE CASE C: Refresh (`/run-pipeline phase:ingest` or `/run-pipeline phase:analyze`)

When new docs are added or decisions are made.

### Phase 1: Incremental Ingestion

Launch `doc-ingester` with prompt:
"Run in INCREMENTAL INGESTION mode. Read docs/.ingestion-manifest.json. Only process new or modified documents. Update docs/PROJECT_CONTEXT.md with changes. Update the manifest."

Report: "{n} new docs, {m} updated docs, {k} unchanged (skipped)"

### Then optionally:

- Run Phase 2 in refresh mode: gap-analyst checks if gaps were resolved by new info
- Run Phase 3 in refresh mode: epic-decomposer checks if Epics need adjustment

---

## USE CASE D: Jira Sync (`/run-pipeline phase:sync`)

Pull Jira state back to docs.

```
Dispatch: jira-sync agent
Input: docs/JIRA_CREATION_REPORT.md + Jira MCP queries
Output: docs/SYNC_REPORT.md
```

1. Launch the `jira-sync` subagent via the Agent tool with prompt:
   "Read docs/JIRA_CREATION_REPORT.md for the local-to-Jira mapping. Query Jira using jira_search for all issues in project {project-key}. Compare against docs/EPICS.md and docs/stories/. Produce docs/SYNC_REPORT.md."
2. Verify `docs/SYNC_REPORT.md` was created
3. Present the sync summary: progress percentages, status changes, new issues, missing issues

**HUMAN CHECKPOINT**: Present sync report.
Ask: "Sync complete. {summary}. Review `docs/SYNC_REPORT.md`. Would you like me to update the local docs to reflect Jira state? Reply 'update-docs' to proceed, or 'skip' to keep docs as-is."

If user approves updates:
- Re-launch `jira-sync` with prompt: "Update the local docs based on the approved SYNC_REPORT.md. Add status badges to stories, update Epic progress in EPICS.md, mark resolved gaps in GAP_ANALYSIS.md."

---

## Error Handling

- If any agent fails, report the error and ask the user how to proceed
- If Jira MCP calls fail, log the error, skip that ticket, and continue with the rest
- At the end, report any skipped tickets so they can be created manually
- Never retry Jira creation automatically — failed tickets need human review

## State Management

### Human-Readable State (Markdown)

| File | Phase | Persists Across Runs |
|------|-------|---------------------|
| `PROJECT_CONTEXT.md` | Phase 1 | Yes (incremental updates) |
| `GAP_ANALYSIS.md` | Phase 2 | Yes (gaps accumulate, get resolved) |
| `EPICS.md` | Phase 3 | Yes (Epics accumulate) |
| `stories/EPIC-{n}-stories.md` | Phase 4 | Yes (stories accumulate per Epic) |
| `VALIDATION_REPORT.md` | Phase 5 | Yes (updated per validation run) |
| `JIRA_CREATION_REPORT.md` | Phase 6 | Yes (mapping grows with each Jira push) |
| `SYNC_REPORT.md` | Sync | Regenerated each sync |
| `archive/*` | Orchestrator | Archived resolved/completed items |

### Machine-Readable State (JSON)

| File | Purpose | Persists |
|------|---------|----------|
| `.pipeline-state.json` | Validation gate, Jira resume, ID registry, checkpoint enforcement | Yes |
| `.ingestion-manifest.json` | Tracks which docs have been read (modification dates + hashes) | Yes |

### Pipeline Properties

The pipeline is **resumable** — if it fails at any phase, re-run that specific phase. Jira creation can resume mid-way without duplicates.
The pipeline is **incremental** — focused runs and refreshes append, not overwrite. IDs are managed via the state file to prevent collisions.
The pipeline is **cyclical** — Discovery → Deep Dives → Refresh → Sync → repeat.
The pipeline is **gated** — human checkpoints are enforced by the Agent PreToolUse hook checking `checkpoint.awaiting_human`.
The pipeline is **self-maintaining** — document health checks archive resolved items when bloat exceeds thresholds.
