# Phase 1: Documentation Ingestion

## When This Runs
Always — after Phase 0 (scale assessment).

## Agent
**Aria** (Doc Ingester) — `.claude/agents/doc-ingester.md`

## Input
- `docs/` directory (all project documentation files)
- `docs/.ingestion-manifest.json` (if exists — for incremental mode)

## Process

### Step 1: Determine ingestion mode

| Condition | Mode |
|-----------|------|
| `docs/PROJECT_CONTEXT.md` does not exist | Full ingestion |
| `docs/PROJECT_CONTEXT.md` exists AND `$ARGUMENTS.phase` is "ingest" | Incremental ingestion |
| `docs/PROJECT_CONTEXT.md` exists AND pipeline mode is "discovery" | Full ingestion |

### Step 2: Verify prerequisites

- Confirm `docs/` contains at least one documentation file (excluding templates, stories,
  generated outputs, state files)
- If no docs found, stop and tell the user

### Step 3: Dispatch Doc Ingester

Launch the `doc-ingester` subagent via the Agent tool with prompt:

**Full mode:**
"Run in FULL INGESTION mode. Read all documents in docs/ and create docs/PROJECT_CONTEXT.md
and docs/.ingestion-manifest.json."

**Incremental mode:**
"Run in INCREMENTAL INGESTION mode. Read docs/.ingestion-manifest.json. Only process new
or modified documents. Update docs/PROJECT_CONTEXT.md with changes. Update the manifest."

### Step 4: Verify output

- Confirm `docs/PROJECT_CONTEXT.md` was created/updated
- Confirm `docs/.ingestion-manifest.json` was created/updated

### Step 5: Context extraction

**Critical**: Read `docs/PROJECT_CONTEXT.md` NOW and store its content for context injection
into subsequent agents. Extract section groups as defined in the orchestrator's
Context Injection Protocol:

- **Core context** (all agents): Sections 1, 8
- **Analysis context** (Gap Analyst): Full file
- **Architecture context** (Epic Decomposer): Sections 1-4, 7, 10
- **Story context** (Story Writer): Sections 1-2, 9

### Step 6: Report

"Phase 1 complete. PROJECT_CONTEXT.md generated from {n} documents.
{In incremental mode: '{new} new, {updated} updated, {skipped} unchanged (skipped)'}"

## Output
- `docs/PROJECT_CONTEXT.md` — structured project context
- `docs/.ingestion-manifest.json` — file tracking manifest
- Extracted context sections (held in orchestrator memory for injection)

## State Updates
```json
{
  "checkpoint": {
    "current_phase": "ingest",
    "last_completed_phase": "assessment"
  }
}
```

## Checkpoint
None — Phase 1 does not have a human checkpoint.
