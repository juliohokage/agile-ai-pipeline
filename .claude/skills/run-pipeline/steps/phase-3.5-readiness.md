# Phase 3.5: Implementation Readiness Gate

## When This Runs
**Thorough mode only.** Skip in lite and full modes.

## Agent
Inline (orchestrator handles this directly, no subagent dispatch).

## Input
- `docs/GAP_ANALYSIS.md` — open gaps
- `docs/EPICS.md` — Epic definitions with dependencies
- `docs/.pipeline-state.json` — ID registry

## Process

### Step 1: Gap coverage check

For every OPEN gap in GAP_ANALYSIS.md:
1. Find which Epics reference this gap
2. Build a coverage matrix: Gap ID → Epic IDs

**If any open gap has zero Epic coverage** → CONCERN: "Gap {ID} ({description}) has no
Epic assigned. Stories cannot be written for uncovered gaps."

### Step 2: Dependency acyclicity check

Parse the dependency graph from EPICS.md:
1. Build a directed graph: Epic → depends on Epics
2. Run cycle detection (topological sort)

**If cycles found** → FAIL: "Circular dependency detected: {cycle path}.
Epic decomposition must be revised."

### Step 3: Priority consistency check

For each dependency relationship:
1. If a "Must Have" Epic depends on a "Could Have" or "Won't Have" Epic → CONCERN:
   "EPIC-{n} (Must Have) is blocked by EPIC-{m} (Could Have). Priority mismatch —
   either raise EPIC-{m}'s priority or remove the dependency."

### Step 4: Scope reasonableness check

Count total estimated stories across all Epics:
- If > 100 stories total → CONCERN: "Scope is very large ({n} estimated stories).
  Consider running focused deep dives per domain instead of writing all stories at once."
- If any single Epic estimates > 8 stories → CONCERN: "EPIC-{n} estimates {m} stories.
  Consider splitting into sub-Epics."

### Step 5: Generate result

| Outcome | Condition |
|---------|-----------|
| **PASS** | All gaps covered, no cycles, no priority mismatches, reasonable scope |
| **CONCERNS** | Issues found but none are blocking (priority mismatches, large scope) |
| **FAIL** | Cycles detected or gaps with zero coverage |

Present the result:

```
"Implementation Readiness Gate: {PASS|CONCERNS|FAIL}

{If CONCERNS or FAIL, list each issue with its category}

Reply 'proceed' to continue to story writing, 'fix' to address the issues first,
or 'skip-gate' to override and proceed anyway."
```

## Output
- No file output — this is an inline check
- Result logged to `docs/.pipeline-state.json`

## State Updates
```json
{
  "readiness_gate": {
    "status": "{PASS|CONCERNS|FAIL}",
    "issues": ["{issue descriptions}"],
    "timestamp": "{ISO-8601}",
    "overridden": {true|false}
  }
}
```

## Checkpoint
Inline with the gate result — user confirms proceed, fix, or override.
This is NOT a progressive checkpoint — it's a pass/fail gate.
