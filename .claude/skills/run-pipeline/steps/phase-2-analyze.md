# Phase 2: Gap Analysis

## When This Runs
Always — after Phase 1 (ingestion).

## Agent
**Sophia** (Gap Analyst) — `.claude/agents/gap-analyst.md`

## Input
- `docs/PROJECT_CONTEXT.md` (injected — full content, analysis context)
- `docs/GAP_ANALYSIS.md` (if exists — for focused/refresh modes)
- ID registry from `docs/.pipeline-state.json` → `ids.highest_gap`

## Process

### Step 1: Determine analysis mode

| Pipeline Mode | Gap Analysis Mode |
|--------------|-------------------|
| Discovery | Full analysis |
| Focused | Focused analysis (scoped to `$ARGUMENTS.focus`) |
| Refresh | Refresh analysis (check if gaps were resolved by new info) |

### Step 2: Dispatch Gap Analyst

Launch the `gap-analyst` subagent via the Agent tool with prompt that includes:

**Full mode:**
"Run in FULL ANALYSIS mode. Create docs/GAP_ANALYSIS.md."

**Focused mode:**
"Run in FOCUSED ANALYSIS mode. Focus topic: '{refined focus}'.
Read existing docs/GAP_ANALYSIS.md. Only analyze gaps related to '{refined focus}'.
Append new gaps starting from ID {next_gap_id}."

**Refresh mode:**
"Run in REFRESH mode. Read existing docs/GAP_ANALYSIS.md. Check if any gaps have been
resolved by the updated PROJECT_CONTEXT.md. Mark resolved gaps accordingly."

**Always inject:** Full PROJECT_CONTEXT.md content as `## Project Context (injected)` block.

Consider using `superpowers:brainstorming` if the analysis needs creative exploration
(e.g., the topic is vague or the docs are thin).

### Step 3: Verify output

- Confirm `docs/GAP_ANALYSIS.md` was created/updated
- Extract gap counts: total, by type (Requirement, Knowledge, Decision, Resource, Technical, Process),
  by severity (Critical, High, Medium, Low)

### Step 4: Update state

Update `docs/.pipeline-state.json`:
- `ids.highest_gap` → highest gap ID produced
- `checkpoint.current_phase` → "analyze"
- `checkpoint.awaiting_human` → `true`

## Output
- `docs/GAP_ANALYSIS.md` — gap inventory with risk matrix

## State Updates
```json
{
  "ids": { "highest_gap": {n} },
  "checkpoint": {
    "current_phase": "analyze",
    "awaiting_human": true
  }
}
```

## Checkpoint

**HUMAN CHECKPOINT** — mode-dependent:

### lite / full mode:
"Gap analysis complete. {n} gaps identified ({critical} critical, {high} high priority).
Please review `docs/GAP_ANALYSIS.md`. Reply 'proceed' to continue to Epic decomposition,
or provide feedback."

### thorough mode (progressive):

**Level 1 — Orientation:**
"{n} gaps identified. {critical} critical, {high} high priority.
Gap types: {count per type}. Domains affected: {list}."

**Level 2 — Walkthrough:**
Present key gaps organized by domain (not by ID order). Each gap explained with its impact.
Group related gaps together.

**Level 3 — Detail Pass:**
Top 3 highest-risk gaps highlighted with:
- Specific stakeholder questions that would resolve them
- Tagged by category: `[architecture]`, `[security]`, `[scope]`, `[compliance]`

After user responds: set `checkpoint.awaiting_human` to `false`,
set `checkpoint.last_approved_phase` to "analyze".
