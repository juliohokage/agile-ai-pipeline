# Phase 0: Scale Assessment

## When This Runs
Always — before Phase 1. This is the first phase of every pipeline run.

## Agent
Inline (orchestrator handles this directly, no subagent dispatch).

## Input
- Quick scan of all files in `docs/` (excluding generated outputs, templates, stories)
- Read file sizes, count distinct terms, scan for TBDs and risk indicators
- Do NOT perform full ingestion — this is a lightweight pre-scan

## Process

### Step 1: Scan documents

Read all documentation files in `docs/` (excluding `docs/stories/`, `docs/templates/`,
`docs/archive/`, `docs/decisions/`, and generated files like PROJECT_CONTEXT.md,
GAP_ANALYSIS.md, EPICS.md, VALIDATION_REPORT.md, SYNC_REPORT.md, JIRA_CREATION_REPORT.md).

For each file, extract:
- Word count
- Distinct system/component/service names mentioned
- Count of TBD/TODO/unclear/unknown markers
- Count of contradictions (same term defined differently)
- Domain keywords (infrastructure, security, UX, data, compliance, etc.)
- Regulatory/compliance mentions (SOC2, HIPAA, GDPR, PCI, etc.)

### Step 2: Score across 6 dimensions

| Dimension | What it measures | Score 1-5 |
|-----------|-----------------|-----------|
| **Volume** | Total word count across all docs | 1=<2K words, 2=2-10K, 3=10-25K, 4=25-50K, 5=>50K |
| **Complexity** | Distinct systems, components, services mentioned | 1=monolith/1-2 components, 2=3-5, 3=6-10, 4=11-20, 5=>20 |
| **Clarity** (inverted) | Ratio of explicit decisions vs TBDs/TODOs/unclear items | 1=all decided (<5% TBD), 2=few TBDs (5-15%), 3=some (15-30%), 4=many TBDs (30-50%), 5=mostly TBD (>50%) |
| **Consistency** (inverted) | Contradictions detected between docs | 1=fully aligned (0), 2=few conflicts (1-2), 3=some (3-4), 4=several (5-10), 5=many conflicts (>10) |
| **Domain breadth** | Number of distinct domains (infra, security, UX, data, etc.) | 1=single domain, 2=2 domains, 3=3 domains, 4=4 domains, 5=5+ domains |
| **Risk indicators** | Regulatory, compliance, security mentions | 1=none, 2=mentioned once, 3=several mentions, 4=dedicated sections, 5=heavily regulated |

**Note on Clarity and Consistency**: These are scored inversely — a project with many TBDs
and contradictions needs MORE pipeline depth, not less. The rubric above already accounts
for this (higher raw score = more work needed), so the composite formula is a direct sum.

### Step 3: Compute composite score

```
composite = volume + complexity + clarity + consistency + domain_breadth + risk_indicators
```

Score range: 6-30.

### Step 4: Map to mode

Read `config/scale-thresholds.md` for current threshold values, then:

| Composite Score | Recommended Mode |
|----------------|-----------------|
| 6-12 | **lite** |
| 13-20 | **full** |
| 21-30 | **thorough** |

### Step 5: Present to user and get confirmation

```
"Quick scan complete.
  Project complexity score: {score}/30
  Breakdown:
    Volume: {n}/5 ({word_count} words across {file_count} docs)
    Complexity: {n}/5 ({component_count} distinct components)
    Clarity: {n}/5 ({tbd_count} TBDs found)
    Consistency: {n}/5 ({contradiction_count} contradictions)
    Domain breadth: {n}/5 ({domain_count} domains)
    Risk indicators: {n}/5 ({risk_detail})
  Recommended mode: {mode}

  [proceed] [override to: lite/full/thorough]"
```

Wait for user response. If they override, use their chosen mode.

If `$ARGUMENTS.mode` was explicitly provided, skip the recommendation and use that mode directly.
Still show the score for informational purposes.

## Output
- Mode selection stored in `docs/.pipeline-state.json` → `scale.score` and `scale.mode`

## State Updates
```json
{
  "scale": {
    "score": {composite_score},
    "mode": "{lite|full|thorough}",
    "dimensions": {
      "volume": {n},
      "complexity": {n},
      "clarity": {n},
      "consistency": {n},
      "domain_breadth": {n},
      "risk_indicators": {n}
    },
    "timestamp": "{ISO-8601}",
    "overridden": {true|false}
  }
}
```

## Checkpoint
User confirmation of mode selection (or override). This is NOT a progressive checkpoint —
it's a simple confirm/override interaction.
