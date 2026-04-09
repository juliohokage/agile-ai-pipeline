# Phase 3: Epic Decomposition

## When This Runs
Always — after Phase 2 (gap analysis) and Phase 2.5 (party mode, if thorough).

## Agent
**Marcus** (Epic Decomposer) — `.claude/agents/epic-decomposer.md`

## Input
- `docs/PROJECT_CONTEXT.md` (injected — architecture context: sections 1-4, 7, 10)
- `docs/GAP_ANALYSIS.md` (full file)
- `docs/EPICS.md` (if exists — for focused/refresh modes)
- ID registry from `docs/.pipeline-state.json` → `ids.highest_epic`
- Jira state awareness (if `jira_creation.created` is non-empty or `docs/SYNC_REPORT.md` exists)

## Process

### Step 1: Determine decomposition mode

| Pipeline Mode | Epic Decomposition Mode |
|--------------|------------------------|
| Discovery | Full decomposition |
| Focused | Focused — create/extend Epics for focus topic only |
| Refresh | Refresh — check if Epics need adjustment |

### Step 2: Jira state awareness (focused/refresh only)

If `docs/.pipeline-state.json` → `jira_creation.created` is non-empty,
or if `docs/SYNC_REPORT.md` exists:
- Read the Jira creation state to know which Epics have been pushed and their current status
- Include in the subagent prompt: "The following Epics already exist in Jira: {list with status}.
  Do not create work that duplicates completed Epics. New Epics may depend on existing ones —
  reference them by number."

### Step 3: Dispatch Epic Decomposer

Launch the `epic-decomposer` subagent via the Agent tool with prompt:

**Full mode:**
"Run in FULL DECOMPOSITION mode. Read docs/GAP_ANALYSIS.md. Create docs/EPICS.md.
Follow the template in docs/templates/epic-template.md."

**Focused mode:**
"Run in FOCUSED DECOMPOSITION mode. Focus topic: '{refined focus}'.
Read docs/GAP_ANALYSIS.md and existing docs/EPICS.md. Create or extend Epics ONLY for
'{refined focus}'. Start numbering from EPIC-{next_epic_number}."

**Always inject:** Architecture context (sections 1-4, 7, 10 from PROJECT_CONTEXT.md)
as `## Project Context (injected)` block.

### Step 4: Verify output

- Confirm `docs/EPICS.md` was created/updated
- Extract: Epic count, dependency graph, execution order
- **Verify gap coverage map accuracy**:
  1. Count all gap IDs in `docs/GAP_ANALYSIS.md` (use `grep -oE '(RG|KG|DG|TG|PG|RSG)-[0-9]+' docs/GAP_ANALYSIS.md | sort -u | wc -l`)
  2. Count resolved vs open gaps (check Status column for "Resolved")
  3. Count gap IDs in the coverage map table in `docs/EPICS.md` (use `grep -oE '^\| (RG|KG|DG|TG|PG|RSG)-[0-9]+' docs/EPICS.md | sort -u | wc -l`)
  4. Verify: covered + uncovered + resolved = total. If the numbers don't add up, fix the coverage map before proceeding — do NOT approximate or accept the agent's self-reported count

### Step 5: Update state

Update `docs/.pipeline-state.json`:
- `ids.highest_epic` → highest Epic number produced
- `checkpoint.current_phase` → "decompose"
- `checkpoint.awaiting_human` → `true`

## Output
- `docs/EPICS.md` — Epic list with overview table, dependency graph, execution order

## State Updates
```json
{
  "ids": { "highest_epic": {n} },
  "checkpoint": {
    "current_phase": "decompose",
    "awaiting_human": true
  }
}
```

## Checkpoint

**HUMAN CHECKPOINT** — mode-dependent:

### lite / full mode:
"Epic decomposition complete. {n} Epics identified. Please review `docs/EPICS.md`.
Reply 'proceed' to continue to Story writing, or provide adjustments."

### thorough mode (progressive):

**Level 1 — Orientation:**
"{n} Epics created. Dependency graph: {summary}. Execution order: {wave summary}."

**Level 2 — Walkthrough:**
Epics organized by priority wave, explaining sequencing rationale.
Why this order? What are the critical-path Epics?

**Level 3 — Detail Pass:**
- Any Epics with 8+ estimated stories flagged for potential splitting
- Any "Must Have" Epics blocked by "Could Have" Epics flagged for priority review
- Specific questions about scope boundaries

After user responds: set `checkpoint.awaiting_human` to `false`,
set `checkpoint.last_approved_phase` to "decompose".
