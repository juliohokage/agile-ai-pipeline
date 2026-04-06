# Phase 4: Story Writing (Parallelizable)

## When This Runs
Always — after Phase 3 (epic decomposition) and Phase 3.5 (readiness gate, if thorough).

## Agent
**Elena** (Story Writer) — `.claude/agents/story-writer.md`
One invocation per Epic. Use `superpowers:dispatching-parallel-agents` to run in parallel.

## Input
- `docs/PROJECT_CONTEXT.md` (injected — story context: sections 1-2, 9)
- `docs/EPICS.md` — Epic details for the assigned Epic
- `docs/templates/story-template.md` — format reference
- ID registry from `docs/.pipeline-state.json` → `ids.highest_story`
- `$ARGUMENTS.epic` (if specified — only process that one Epic)

## Process

### Step 1: Determine which Epics to process

| Condition | Epics to Process |
|-----------|-----------------|
| `$ARGUMENTS.epic` is set | Only EPIC-{epic} |
| Focused mode | Only new/extended Epics from the focused run |
| Full mode | All Epics in EPICS.md |

### Step 2: Build agent prompts

For each Epic to process, construct a prompt:

```
"You are Elena, the Story Writer. Write stories for EPIC-{n}: {Epic Name}.

## Project Context (injected by orchestrator — do NOT re-read PROJECT_CONTEXT.md)
{sections 1-2, 9 from PROJECT_CONTEXT.md}

## Epic Details
{full Epic description from EPICS.md}

## Instructions
- Read docs/templates/story-template.md for format
- Output to docs/stories/EPIC-{n}-stories.md
- Start story numbering from US-{n}.{next_story_number}
- Start spike numbering from SPIKE-{n}.{next_spike_number}
- Follow INVEST criteria. Self-check before finalizing.
- At least 2 Gherkin scenarios per item."
```

### Step 3: Dispatch in parallel

Use `superpowers:dispatching-parallel-agents` to launch all story-writer agents simultaneously.

If only one Epic: dispatch directly via the Agent tool (no need for parallel skill).

### Step 4: Verify output

For each Epic processed:
- Confirm `docs/stories/EPIC-{n}-stories.md` was created/updated
- Count stories and spikes generated

### Step 5: Report

"Phase 4 complete. Stories generated:
{for each Epic: EPIC-{n}: {stories} stories, {spikes} spikes}
Total: {total_stories} stories, {total_spikes} spikes across {epic_count} Epics."

## Output
- `docs/stories/EPIC-{n}-stories.md` (one file per Epic processed)

## State Updates
```json
{
  "ids": {
    "highest_story": {
      "{epic_number}": {highest_story_number_in_that_epic}
    }
  },
  "checkpoint": {
    "current_phase": "stories"
  }
}
```

## Checkpoint
None — Phase 4 does not have a human checkpoint. Stories go directly to validation (Phase 5).
