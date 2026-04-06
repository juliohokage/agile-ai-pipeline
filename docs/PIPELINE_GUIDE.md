# Agile AI Pipeline — Complete Guide

## What is this?

A multi-agent system that transforms project documentation into structured Jira tickets.
It reads your docs, identifies gaps, decomposes work into Epics and User Stories,
validates them, and pushes to Jira — with human approval at every critical step.

**Key principles:**
- **GitOps-driven**: All documentation lives in the repo, viewable via mkdocs
- **Human-in-the-loop**: Mandatory checkpoints enforced by hooks, not just prompt instructions
- **Resumable**: Jira creation tracks state incrementally — crash mid-way and resume without duplicates
- **Token-optimized**: Orchestrator injects only relevant context sections into each agent
- **Self-maintaining**: Document health monitored, resolved items archived automatically
- **No agent estimation**: Story points and effort sizing are done by humans during sprint planning
- **Scale-adaptive**: Pipeline depth adjusts to project complexity — content-aware scoring, not doc count

---

## How the Pipeline Works (Architecture Deep Dive)

### Three Layers

```
Layer 1: Configuration & Hooks (.claude/settings.json)
  ├── PreToolUse hooks  → Jira safety gate, checkpoint enforcement
  └── PostToolUse hooks → Story invalidation on edit

Layer 2: Agents (.claude/agents/*.md)
  ├── 6 specialized subagents + 2 review agents (adversarial + edge case)
  └── Each with persona, defined inputs/outputs, handoff contracts

Layer 3: Orchestrator (.claude/skills/run-pipeline/)
  ├── Step-file architecture (one file per phase)
  ├── Scale-adaptive depth (lite/full/thorough)
  ├── Context injection protocol
  └── State management via .pipeline-state.json
```

### SHA256 Hashing — Why and How

The pipeline uses SHA256 hashes in two critical places:

**1. Ingestion Manifest (`.ingestion-manifest.json`)**
Each ingested document gets a SHA256 hash of its content. On re-ingestion, the pipeline compares the stored hash against the current file hash. If they match, the file is skipped — even if the modification date changed (e.g., due to `git checkout`). This prevents wasted token spend re-reading unchanged content.

```bash
sha256sum docs/ARCHITECTURE_OVERVIEW.md | cut -d' ' -f1
# → "abc123..." stored in manifest, compared on next run
```

**2. Validation Gate (`.pipeline-state.json` → `validation.stories_hash`)**
After validation passes, the orchestrator computes a composite hash of ALL story files:

```bash
find docs/stories -name "EPIC-*-stories.md" -exec sha256sum {} \; | sort | sha256sum
```

This hash is stored in the state file. The Jira creation hook (PreToolUse) re-computes this hash before every Jira API call. If any story file changed since validation, the hash won't match → Jira creation is **blocked**. This guarantees you can never push unvalidated or modified stories to Jira.

**Why SHA256 and not modification dates?** Modification dates are unreliable — git operations, file copies, and editor auto-saves can change mtime without changing content. SHA256 is content-addressed: if the bytes didn't change, the hash is the same. This is the same principle behind Git's own object model.

### The Orchestrator — Step-File Architecture

The orchestrator is decomposed into per-phase step files for token efficiency and maintainability:

```
.claude/skills/run-pipeline/
├── SKILL.md                    ← Entry point: mode detection, scale assessment, routing
├── steps/
│   ├── phase-0-assessment.md   ← Content-aware scoring, mode recommendation
│   ├── phase-1-ingest.md       ← Doc Ingester dispatch + context extraction
│   ├── phase-2-analyze.md      ← Gap Analyst dispatch + party mode (if thorough)
│   ├── phase-2.5-party.md      ← Party mode review (optional, mode-dependent)
│   ├── phase-3-decompose.md    ← Epic Decomposer dispatch
│   ├── phase-3.5-readiness.md  ← Implementation readiness gate (optional)
│   ├── phase-4-stories.md      ← Story Writer dispatch (parallel per Epic)
│   ├── phase-5-validate.md     ← Story Validator dispatch
│   ├── phase-5.5-review.md     ← Adversarial review + Edge case hunter
│   ├── phase-6-jira.md         ← Jira creation with resume support
│   └── phase-sync.md           ← Jira sync agent dispatch
└── config/
    └── scale-thresholds.md     ← Scoring rubric and mode thresholds
```

**Just-in-time loading**: The orchestrator loads only the current step file, never future ones. This keeps context window pressure low and makes each phase independently testable.

**State tracking**: Progress is persisted in `.pipeline-state.json` → `checkpoint.current_phase`. If the pipeline crashes, re-running picks up from the last completed phase.

### Context Injection Protocol

To avoid every subagent re-reading the full `docs/PROJECT_CONTEXT.md` (which wastes tokens), the orchestrator reads it ONCE after Phase 1 and injects only the relevant sections into each subagent's prompt:

| Agent | Sections Injected | Why |
|-------|-------------------|-----|
| Gap Analyst (Sophia) | ALL sections | Needs full project understanding to find gaps |
| Epic Decomposer (Marcus) | 1-4, 7, 10 (Overview, Architecture, Stakeholders, Decisions, Constraints, Open Items) | Needs structure and boundaries, not domain details |
| Story Writer (Elena) | 1-2, 9 (Overview, Architecture, Domain Model) + specific Epic details | Needs enough to write stories, not the whole project |
| Story Validator (Kai) | None — reads story files directly | Validates format and quality, not project content |
| Adversarial Reviewer (Rex) | Story content + GAP_ANALYSIS.md summary | Needs to check story coverage against gaps |
| Edge Case Hunter | Story content only | Mechanically traces paths in Gherkin scenarios |

### Document Compression / Distillation

For large projects where PROJECT_CONTEXT.md grows beyond 50KB, the orchestrator applies **lossless compression** before context injection:

1. **Analyze** — identify information density per section
2. **Compress** — convert prose to dense bullet-point format, strip decorative formatting
3. **Verify** — check all original facts are preserved (round-trip test)

This is especially important for Phase 2 (Gap Analysis) which receives the full context. A 50KB doc compressed to 20KB at 2.5:1 ratio saves ~7,500 tokens per agent call.

The orchestrator uses `superpowers:verification-before-completion` to verify compression losslessness.

### Hook Enforcement (Safety Gates)

Three hooks in `.claude/settings.json` enforce pipeline integrity:

**1. Jira Validation Gate (PreToolUse)**
- **Triggers on**: `jira_create_issue`, `jira_batch_create_issues`, `jira_link_to_epic`, `jira_create_issue_link`
- **Checks**: State file exists → validation status is "PASS" → stories hash matches current files
- **If any check fails**: Blocks the API call with an error message
- **Why hooks, not prompts**: An LLM can be convinced to ignore a prompt instruction. A bash hook **cannot be bypassed** — it runs before the tool call reaches the API.

**2. Checkpoint Enforcement (PreToolUse)**
- **Triggers on**: Any `Agent` tool dispatch
- **Checks**: `checkpoint.awaiting_human` is not `true`
- **Purpose**: Prevents the LLM from skipping human review steps by dispatching the next agent before the user responds

**3. Story Invalidation (PostToolUse)**
- **Triggers on**: Any `Write` or `Edit` to files in `docs/stories/`
- **Action**: Sets `validation.status` to "STALE" in state file
- **Purpose**: If you manually edit a story after validation, the Jira gate will block until you re-validate

---

## Agent Roster

Each agent has a defined persona that biases its output toward the behavior we want. Personas are not cosmetic — they shift the model's attention and output distribution.

| Agent | Persona | Role | Bias |
|-------|---------|------|------|
| **Doc Ingester** | **Aria** — methodical archivist, obsessive about traceability | Reads docs, produces PROJECT_CONTEXT.md | Every fact traceable to a source. Flags conflicts, never resolves them |
| **Gap Analyst** | **Sophia** — paranoid strategist, assumes gaps are worse than they look | Identifies gaps, risks, unknowns | Finds more gaps, higher severity. Challenges "TBD" entries aggressively |
| **Epic Decomposer** | **Marcus** — pragmatic architect, hates scope creep | Creates Epics with boundaries and dependencies | Tight scope per Epic. Explicit "Out of Scope" for every Epic |
| **Story Writer** | **Elena** — meticulous PO, self-polices INVEST criteria | Writes User Stories and Spikes with Gherkin AC | Rejects her own stories if they fail INVEST. Splits XL stories proactively |
| **Story Validator** | **Kai** — strict QA, no mercy for vague AC | Validates stories, gates Jira creation | Fails "should be fast" immediately. Demands measurable criteria |
| **Adversarial Reviewer** | **Rex** — cynical senior dev who has seen projects fail | Finds what's missing, not just what's wrong | Must find minimum 10 issues. Assumes problems exist and searches for them |
| **Edge Case Hunter** | **Nova** — mechanical path tracer, zero opinion, pure logic | Finds unhandled branches in Gherkin scenarios | Enumerates every Given/When/Then path. Reports only unguarded paths |
| **Jira Sync** | **Dash** — project sync specialist, read-only on Jira | Pulls Jira state back to docs | Never modifies Jira. Flags discrepancies factually |

---

## Scale-Adaptive Depth

### The Problem

A project with 3 crystal-clear docs covering a complex microservices system needs thorough analysis. A project with 20 repetitive meeting notes about a simple feature does not. **Doc count is a bad metric for complexity.**

### Content-Aware Scoring (Phase 0)

Before Phase 1, the orchestrator performs a quick scan (not full ingestion) of all docs and scores the project across 6 dimensions:

| Dimension | What it measures | Score 1-5 |
|-----------|-----------------|-----------|
| **Volume** | Total word count across all docs | 1=<2K words, 5=>50K words |
| **Complexity** | Distinct systems, components, services mentioned | 1=monolith, 5=many microservices |
| **Clarity** | Ratio of explicit decisions vs TBDs/TODOs/unclear items | 1=mostly TBD, 5=all decided |
| **Consistency** | Contradictions detected between docs | 1=many conflicts, 5=fully aligned |
| **Domain breadth** | Number of distinct domains (infra, security, UX, data, etc.) | 1=single domain, 5=5+ domains |
| **Risk indicators** | Regulatory, compliance, security mentions | 1=none, 5=heavily regulated |

### Mode Selection

| Score | Mode | What runs | What's skipped |
|-------|------|-----------|----------------|
| 6-12 | **lite** | Phases 1-6 (collapsed: phases 2+3 in one call). Simple checkpoints. INVEST validation only. | Party mode, readiness gate, adversarial review, edge case hunter, progressive checkpoints |
| 13-20 | **full** (default) | All phases. Standard checkpoints. Adversarial review. | Party mode, readiness gate, edge case hunter |
| 21-30 | **thorough** | All phases + Party Mode (2.5) + Readiness Gate (3.5) + Adversarial Review AND Edge Case Hunter (5.5) + Progressive Checkpoints | Nothing — full ceremony |

### User Override

The orchestrator always shows the assessment and lets the user override:

```
Orchestrator: "Quick scan complete.
  Project complexity: HIGH (score: 23/30)
  - 1 doc, but covers 8 microservices with OTLP, Kafka, and Kubernetes
  - 3 TBDs found, 0 contradictions
  - Regulatory mentions: SOC2 compliance referenced
  Recommended mode: thorough

  [proceed] [override to: lite/full]"
```

You can also force a mode: `/run-pipeline mode:lite` or `/run-pipeline mode:thorough`.

### Threshold Calibration Note

The scoring thresholds (6-12, 13-20, 21-30) are initial defaults based on our analysis of the first projects run through this pipeline. **They will need calibration over time.** If you find that "full" mode is too heavy for your projects, adjust the boundaries. The orchestrator logs the score and mode selection in `.pipeline-state.json` so you can review past decisions and tune accordingly.

---

## Pipeline Phases (Full Reference)

### Phase 0: Scale Assessment
**Runs**: Always (before Phase 1)
**Agent**: Orchestrator (inline, no subagent)
**Output**: Mode selection (lite/full/thorough) stored in `.pipeline-state.json`
**Skills used**: None
**Human interaction**: Shows score, asks for confirmation or override

### Phase 1: Documentation Ingestion
**Runs**: Always
**Agent**: Aria (Doc Ingester)
**Input**: `docs/` directory
**Output**: `docs/PROJECT_CONTEXT.md` + `docs/.ingestion-manifest.json`
**Modes**: Full (first run) or Incremental (re-ingestion of new/modified docs only)
**Skills used**: None
**After completion**: Orchestrator reads PROJECT_CONTEXT.md and extracts sections for context injection

### Phase 2: Gap Analysis
**Runs**: Always
**Agent**: Sophia (Gap Analyst)
**Input**: PROJECT_CONTEXT.md (injected, full content)
**Output**: `docs/GAP_ANALYSIS.md`
**Modes**: Full, Focused (scoped to topic, appends), Refresh (marks resolved gaps)
**Skills used**: `superpowers:brainstorming` (if analysis needs creative exploration)

**HUMAN CHECKPOINT** (progressive in thorough mode):
- **Orientation**: "{n} gaps identified. {critical} critical, {high} high priority. Gap types breakdown."
- **Walkthrough**: Key gaps organized by domain, not by ID order. Each gap explained with impact.
- **Detail pass**: Top 3 highest-risk gaps highlighted with specific stakeholder questions.
- In lite/full mode: simplified "Review GAP_ANALYSIS.md. Reply 'proceed' or provide feedback."

### Phase 2.5: Party Mode Review (thorough mode only)
**Runs**: Only in thorough mode
**Agent**: Multi-agent discussion — Sophia (Gap Analyst) + Marcus (Epic Decomposer) + Rex (Adversarial Reviewer)
**Input**: GAP_ANALYSIS.md
**Output**: Appended findings to GAP_ANALYSIS.md → `### Multi-Agent Review (Added: {date})`
**Purpose**: Surfaces strategic gaps the single-agent pass missed. Agents debate gap completeness, priorities, and missing domains.
**Skills used**: `superpowers:dispatching-parallel-agents` (to spawn the multi-agent discussion)

### Phase 3: Epic Decomposition
**Runs**: Always
**Agent**: Marcus (Epic Decomposer)
**Input**: PROJECT_CONTEXT.md (sections 1-4, 7, 10 injected) + GAP_ANALYSIS.md
**Output**: `docs/EPICS.md`
**Modes**: Full, Focused (appends), Refresh
**Skills used**: None
**Contract**: Every open gap must map to at least one Epic. Dependencies must be acyclic. Epic numbers are permanent.

**HUMAN CHECKPOINT** (progressive in thorough mode):
- **Orientation**: "{n} Epics created. Dependency graph overview. Execution order summary."
- **Walkthrough**: Epics organized by priority wave, explaining sequencing rationale.
- **Detail pass**: Any Epics with 8+ stories flagged for potential splitting.

### Phase 3.5: Implementation Readiness Gate (thorough mode only)
**Runs**: Only in thorough mode
**Agent**: Orchestrator (inline check, no subagent)
**Checks**:
- Do the Epics cover ALL open gaps? (gap coverage map)
- Are dependencies acyclic? (graph validation)
- Is the scope reasonable? (total story estimate vs team capacity signal)
- Are there any "Must Have" Epics blocked by "Could Have" Epics? (priority consistency)
**Output**: PASS / CONCERNS (with specifics) / FAIL
**Purpose**: Catches structural issues before investing tokens in story writing

### Phase 4: Story Writing (Parallelizable)
**Runs**: Always
**Agent**: Elena (Story Writer) — one invocation per Epic
**Input**: PROJECT_CONTEXT.md (sections 1-2, 9 injected) + Epic details from EPICS.md
**Output**: `docs/stories/EPIC-{n}-stories.md` (one file per Epic)
**Skills used**: `superpowers:dispatching-parallel-agents` (to run story writers in parallel across Epics)
**Contract**: Every story passes INVEST. Every spike is time-boxed with defined output. At least 2 Gherkin scenarios per item.

### Phase 5: Story Validation
**Runs**: Always
**Agent**: Kai (Story Validator)
**Input**: All `docs/stories/EPIC-*-stories.md` files
**Output**: `docs/VALIDATION_REPORT.md`
**Modes**: Full, Subset (after focused runs), Re-validation (after fixes)
**Skills used**: None
**Gate**: "Ready for Jira: YES/NO" — NO blocks Phase 6

**If validation FAILS**: Orchestrator reports failures and asks: "Auto-fix (re-run story writer with feedback) or manual fix?"

### Phase 5.5: Adversarial Review + Edge Case Hunting
**Runs**: In full mode (adversarial only) and thorough mode (both)
**Agents**: Rex (Adversarial Reviewer) + Nova (Edge Case Hunter, thorough only)

**Adversarial Review (Rex)**:
- Reads all story files + GAP_ANALYSIS.md summary
- Assumes problems exist, searches for them
- Must find minimum 10 issues or re-analyzes deeper
- Checks: completeness gaps, vague requirements, hidden assumptions, missing error scenarios, contradictions with gap analysis
- Output: Findings appended to VALIDATION_REPORT.md → `### Adversarial Review Findings`

**Edge Case Hunter (Nova)** (thorough mode only):
- Reads all Gherkin acceptance criteria mechanically
- Enumerates every Given/When/Then path
- Derives edge classes: missing else/default, unguarded inputs, off-by-one, boundary conditions, empty states, concurrent scenarios
- Reports only unhandled paths — silently discards handled ones
- Output: JSON findings appended to VALIDATION_REPORT.md → `### Edge Case Analysis`

**HUMAN CHECKPOINT** (progressive):
- **Orientation**: "Validation {PASSED/FAILED}. Adversarial review found {n} issues. Edge case hunter found {m} unhandled paths."
- **Walkthrough**: Issues organized by severity, not by story order.
- **Detail pass**: Top 5 highest-impact findings highlighted.
- **Decision**: "Reply 'create-jira' to push to Jira, 'fix' to address findings first, or 'skip' to stop here."

**Skills used**: `superpowers:verification-before-completion` (pre-flight checks before Jira creation decision)

### Phase 6: Jira Creation
**Runs**: Only if user explicitly requests it
**Agent**: Orchestrator (direct Jira MCP calls, no subagent)
**Input**: Validated stories + EPICS.md
**Output**: `docs/JIRA_CREATION_REPORT.md` + Jira tickets
**Tools**: `mcp__atlassian__jira_create_issue`, `jira_link_to_epic`, `jira_create_issue_link`, `jira_search`
**Resume support**: State file tracks each created ticket. Re-running skips already-created items.

**Pre-flight checks** (uses `superpowers:verification-before-completion`):
1. `.pipeline-state.json` → `validation.status` == "PASS"
2. Stories hash matches (no modifications since validation)
3. User has explicitly approved Jira creation
4. Project key is configured (not "ADDPROJECTKEY")

### Phase: Jira Sync
**Runs**: On demand (`/run-pipeline phase:sync`)
**Agent**: Dash (Jira Sync)
**Input**: `docs/JIRA_CREATION_REPORT.md` + Jira MCP queries
**Output**: `docs/SYNC_REPORT.md`
**Contract**: Read-only on Jira — never modifies issues. Doc updates only after human approval.

---

## Progressive Checkpoints (Replacing "Review and Proceed")

Traditional checkpoints dump a big markdown file and say "review and proceed." This fails because reviewers either skim (missing issues) or read exhaustively (losing the thread).

Progressive checkpoints guide the reviewer through three levels:

### Level 1: Orientation (always shown)
- One-line summary of what happened
- Key numbers: counts, severity breakdown
- Surface area: what was touched, what wasn't

### Level 2: Walkthrough (shown on request or in thorough mode)
- Findings organized by **concern/domain**, not by file/ID order
- Each concern explained with *why it matters*
- Sequenced so you never encounter a reference to something you haven't seen yet

### Level 3: Detail Pass (shown on request or in thorough mode)
- 3-5 highest blast-radius items
- Specific questions that, if answered, would resolve the biggest risks
- Tagged by category: `[architecture]`, `[security]`, `[scope]`, `[compliance]`

In **lite mode**: Only orientation is shown. User can ask for more detail.
In **full mode**: Orientation + detail pass for critical items.
In **thorough mode**: All three levels presented sequentially.

---

## Course Correction Workflow

When a focused deep-dive or Jira sync reveals that existing Epics need significant changes — not just appending new work, but re-prioritizing, re-scoping, or splitting existing Epics — the pipeline supports a formal course correction:

**Trigger**: User says "correct course" or orchestrator detects:
- A sync report showing >30% of stories in an Epic were descoped or deprioritized in Jira
- A focused run producing gaps that contradict existing Epic assumptions
- Human feedback at a checkpoint requesting structural changes

**Process**:
1. **Assess impact**: What existing Epics are affected? Which stories need to change?
2. **Propose changes**: Re-prioritization, Epic splitting, scope adjustment, dependency rewiring
3. **Stakeholder communication**: Generate a change summary suitable for sharing with the team
4. **Apply changes**: Update EPICS.md, story files, and GAP_ANALYSIS.md with the corrections
5. **Re-validate**: Invalidate the validation gate, requiring re-validation before any Jira push

**Important**: Course correction modifies existing docs (unlike focused runs which only append). It preserves audit trail by marking changes with `⚠️ COURSE CORRECTION ({date}): {reason}`.

---

## Files Overview

### Created by you (inputs)

| File | Purpose |
|------|---------|
| `docs/ARCHITECTURE_OVERVIEW.md` | Your system architecture document |
| `docs/*.md` | Any other project docs (specs, meeting notes, RFCs, etc.) |
| `docs/meeting-transcripts/{YYYY-MM-DD}-{topic}.md` | Meeting transcripts for ingestion |

### Generated by the pipeline (outputs)

| File | Generated By | Purpose |
|------|-------------|---------|
| `docs/PROJECT_CONTEXT.md` | Aria — Doc Ingester (Phase 1) | Structured summary of all documentation |
| `docs/GAP_ANALYSIS.md` | Sophia — Gap Analyst (Phase 2) | Current vs desired state, risks, open questions |
| `docs/EPICS.md` | Marcus — Epic Decomposer (Phase 3) | Epic list with priorities, dependencies, sequencing |
| `docs/stories/EPIC-{n}-stories.md` | Elena — Story Writer (Phase 4) | User Stories and Spikes per Epic |
| `docs/VALIDATION_REPORT.md` | Kai — Story Validator (Phase 5) + Rex + Nova (Phase 5.5) | Quality gate — pass/fail per story + adversarial findings |
| `docs/JIRA_CREATION_REPORT.md` | Orchestrator (Phase 6) | Mapping of local IDs to Jira keys |
| `docs/SYNC_REPORT.md` | Dash — Jira Sync | Diff between Jira state and local docs |
| `docs/decisions/*.md` | Created from Spikes | Decision documents from research tasks |
| `docs/archive/*.md` | Orchestrator | Archived resolved/completed items |

### State files (machine-readable)

| File | Purpose |
|------|---------|
| `docs/.pipeline-state.json` | Pipeline integrity: validation gate, Jira resume, ID registry, checkpoint enforcement, scale score |
| `docs/.ingestion-manifest.json` | Tracks which docs have been ingested (modification dates + SHA256 hashes) |

### Configuration files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Project context for Claude Code — read on every conversation start |
| `.claude/settings.json` | Hooks (Jira safety gate, checkpoint enforcement, story invalidation) |
| `.claude/agents/*.md` | 8 specialized agent definitions (6 pipeline + 2 review) |
| `.claude/skills/run-pipeline/SKILL.md` | Orchestrator entry point |
| `.claude/skills/run-pipeline/steps/*.md` | Per-phase step files (step-file architecture) |
| `docs/templates/story-template.md` | Template: INVEST story format with Gherkin AC (single source of truth) |
| `docs/templates/epic-template.md` | Template: MoSCoW epic format with dependencies |

---

## Commands

### Full Pipeline

```bash
/run-pipeline
```

Runs Phase 0 (scale assessment) → all phases sequentially. The mode (lite/full/thorough) is determined by content-aware scoring or user override.

### Forced Mode

```bash
/run-pipeline mode:lite        # Minimal ceremony, collapsed phases
/run-pipeline mode:full        # Standard (default for most projects)
/run-pipeline mode:thorough    # Full ceremony with all review gates
```

### Focused Deep Dive

```bash
/run-pipeline focus:"metrics backend — evaluate Mimir vs VictoriaMetrics for ingestion"
/run-pipeline focus:"alerts and remediation"
```

Scoped analysis on a specific topic. Includes:
- **Focus refinement**: If your topic is vague, the orchestrator asks clarifying questions (WHAT, WHY, SCOPE)
- **Staleness check**: Warns if docs changed since last ingestion
- **Cross-run awareness**: Knows which Epics already exist in Jira
- **Appends** to existing files, never overwrites

### Run Individual Phases

```bash
/run-pipeline phase:ingest                       # Re-ingest docs (incremental)
/run-pipeline phase:analyze                      # Re-run gap analysis
/run-pipeline phase:decompose                    # Re-run epic decomposition
/run-pipeline phase:write-stories                # Write stories for all epics
/run-pipeline phase:write-stories epic:3         # Write stories for Epic 3 only
/run-pipeline phase:validate                     # Validate all stories
/run-pipeline phase:create-jira project-key:MYPROJ  # Push to Jira
/run-pipeline phase:sync                         # Pull Jira state back to docs
```

### Quick Story (Standalone)

```bash
/user-story topic:"user authentication via SSO"
/user-story topic:"evaluate Mimir vs VictoriaMetrics" type:spike epic:3
/user-story topic:"retry logic for failed API calls" --validate --jira
```

Creates a single story/spike without running the full pipeline. With `--validate --jira`, it validates and pushes in one flow.

### Status & Health

```bash
/project-status    # Quick local status with recommended next actions
/sprint-health     # Live Jira query for sprint health dashboard
```

---

## The 4 Use Cases

### Use Case A: Full Discovery

**When**: Project kickoff, first time running the pipeline.
**What happens**: Assesses project complexity, reads all docs, analyzes everything, creates complete Epics and Stories.

```
You → put docs in docs/ → /run-pipeline → scale assessment → review at each checkpoint → Jira tickets created
```

### Use Case B: Focused Deep Dive

**When**: You need to go deeper on a specific topic.
**What happens**: Refines your focus topic, checks for stale context, only analyzes the scoped area.

```
You → /run-pipeline focus:"topic" → scope refined → focused gaps → focused epics → stories → Jira
```

**Tip**: Be specific. Instead of `focus:"metrics"`, use `focus:"metrics backend — evaluate Mimir vs VictoriaMetrics for ingestion only"`. If your topic is vague, the orchestrator will ask you to clarify.

### Use Case C: Refresh

**When**: New docs were added, decisions were made, or the project evolved.
**What happens**: Incrementally re-reads only changed docs, updates PROJECT_CONTEXT.md.

```
You → add/update docs → /run-pipeline phase:ingest → optionally re-run analysis
```

### Use Case D: Jira Sync

**When**: After sprint work, to see what moved in Jira.
**What happens**: Queries Jira, compares with local docs, produces a diff report.

```
You → /run-pipeline phase:sync → review sync report → approve doc updates
```

---

## The Pipeline Lifecycle

```
    ┌──────────────────────────────────────────────┐
    │                                              │
    ▼                                              │
Full Discovery (A)                                 │
    │                                              │
    ├──► Deep Dives (B) ──► Deep Dives (B) ...     │
    │                                              │
    ├──► Course Correction (if needed) ────────────┤
    │                                              │
    ▼                                              │
Jira Creation (Phase 6) — or skip                  │
    │                                              │
    ▼                                              │
 ┌─────────────────────────────────┐               │
 │  SPRINT WORK HAPPENS            │               │
 │  (team works in Jira daily)     │               │
 └──────────────┬──────────────────┘               │
                │                                  │
                ▼                                  │
 Jira Sync (D) ─── new gaps found? ───► Deep Dive (B)
                │                                  │
                ├── new docs added? ───► Refresh (C) ──┘
                │
                └── sprint complete ───► next sprint planning
```

---

## Safety Gates & Integrity

### 1. Jira Validation Gate (PreToolUse Hook)

**Trigger**: Any Jira creation/linking tool call
**Mechanism**: Reads `docs/.pipeline-state.json`:
- Checks `validation.status` is "PASS"
- Computes current SHA256 of all story files, compares against `validation.stories_hash`
- If stories were modified since validation, blocks with "STALE" error
**Purpose**: Prevents pushing unvalidated or modified stories to Jira
**Why a hook, not a prompt**: Hooks are enforced by bash — the LLM cannot bypass them regardless of prompt.

### 2. Human Checkpoint Enforcement (PreToolUse Hook)

**Trigger**: Any Agent tool dispatch
**Mechanism**: Reads `docs/.pipeline-state.json` → `checkpoint.awaiting_human`
- If `true`, blocks agent dispatch with "awaiting human approval" message
**Purpose**: Prevents the LLM from skipping human review steps

### 3. Story Invalidation (PostToolUse Hook)

**Trigger**: Any Write or Edit to files in `docs/stories/`
**Mechanism**: Sets `validation.status` to "STALE" in `docs/.pipeline-state.json`
**Purpose**: Automatically invalidates validation when stories change, requiring re-validation

### 4. Resumable Jira Creation

If Jira creation crashes mid-way, `docs/.pipeline-state.json` → `jira_creation.created` contains all successfully created tickets. On retry, the orchestrator skips already-created items. Each ticket is recorded **immediately** after creation, not batched.

### 5. ID Registry

Gap IDs, Epic numbers, and story numbers are tracked in `docs/.pipeline-state.json` → `ids`. The orchestrator passes the next available ID range to each agent, preventing collisions across focused runs. IDs are permanent — never reused, even for resolved items.

---

## Mode Comparison Matrix

| Feature | lite | full (default) | thorough |
|---------|------|----------------|----------|
| Phase 0 (Scale Assessment) | Yes | Yes | Yes |
| Phase 1 (Ingestion) | Yes | Yes | Yes |
| Phase 2 (Gap Analysis) | Yes | Yes | Yes |
| Phase 2.5 (Party Mode) | -- | -- | Yes |
| Phase 3 (Epic Decomposition) | Yes (collapsed with Phase 2) | Yes | Yes |
| Phase 3.5 (Readiness Gate) | -- | -- | Yes |
| Phase 4 (Story Writing) | Yes | Yes | Yes |
| Phase 5 (INVEST Validation) | Yes | Yes | Yes |
| Phase 5.5 (Adversarial Review) | -- | Yes | Yes |
| Phase 5.5 (Edge Case Hunter) | -- | -- | Yes |
| Progressive Checkpoints | Orientation only | Orientation + Detail | All 3 levels |
| Phase 6 (Jira Creation) | Yes | Yes | Yes |
| Parallel Story Writing | Yes | Yes | Yes |
| Context Injection | Yes | Yes | Yes + Compression |
| Course Correction | Yes | Yes | Yes |

---

## Integrations

| Integration | MCP Server | Tools Used |
|-------------|-----------|------------|
| **Jira** | `sooperset/mcp-atlassian` | `jira_create_issue`, `jira_batch_create_issues`, `jira_link_to_epic`, `jira_create_issue_link`, `jira_search`, `jira_get_issue` |
| **Context7** | `@upstash/context7-mcp` | Library/framework documentation lookup |

---

## Superpowers Skills Used

The pipeline leverages built-in superpowers skills at specific points:

| Skill | Used In | Purpose |
|-------|---------|---------|
| `superpowers:brainstorming` | Phase 2 (Gap Analysis) | Assists with creative exploration when analysis needs divergent thinking |
| `superpowers:dispatching-parallel-agents` | Phase 4 (Story Writing), Phase 2.5 (Party Mode) | Dispatches multiple story-writer agents in parallel (one per Epic), and spawns multi-agent discussions |
| `superpowers:verification-before-completion` | Phase 5.5 → Phase 6 transition | Pre-flight checks before Jira creation (validation status, hash match, user approval, project key) |

---

## Testing Without Jira

You can run the full pipeline (Phases 0-5.5) without any Jira configuration. At the Phase 5/5.5 human checkpoint, simply say "skip" instead of "create-jira". All local work (context, gaps, epics, stories, validation, adversarial review) will be complete and available in `docs/`.

---

## Setup Instructions

### 1. Copy to your project

```
.claude/                         # Agents, skills, settings, hooks
docs/templates/                  # Story and Epic templates
docs/PIPELINE_GUIDE.md           # This guide
CLAUDE.md                        # Project context for Claude Code
```

### 2. Add your documentation

Place your project docs in `docs/`:
```
docs/ARCHITECTURE_OVERVIEW.md    # Your main architecture doc
docs/meeting-transcripts/*.md    # Meeting transcripts (YYYY-MM-DD-topic.md)
docs/specs/*.md                  # Specifications
docs/rfcs/*.md                   # RFCs or design docs
```

### 3. Configure Jira (optional)

Replace `ADDPROJECTKEY` in `CLAUDE.md` with your actual Jira project key.
Ensure `mcp-atlassian` is configured in your Claude Code MCP settings.

### 4. Run the pipeline

```bash
/run-pipeline                    # First run: full discovery
```

The orchestrator will assess your project, recommend a mode, and guide you through each phase.

### 5. Iterate

```bash
/run-pipeline focus:"specific topic"  # Go deeper on areas
/run-pipeline phase:sync              # After sprint work
/run-pipeline phase:ingest            # After adding new docs
```

---

## FAQ

**Q: Can I run the pipeline without Jira?**
A: Yes. Phases 0-5.5 are entirely local. At the Phase 5.5 checkpoint, say "skip" to stop before Jira creation.

**Q: Can I run the pipeline on a project with no documentation?**
A: No. The pipeline needs at least one doc to ingest. Even a rough architecture sketch or meeting notes will work.

**Q: What if I disagree with the gap analysis or epic structure?**
A: That's what the human checkpoints are for. Provide feedback and the agent will adjust. In thorough mode, you get progressive checkpoints that help you understand findings before approving.

**Q: Can I create Jira tickets without validation?**
A: No. The Jira safety gate hook blocks creation unless validation passes and stories haven't been modified since. This is enforced by a bash hook, not just a prompt instruction.

**Q: What if someone creates tickets directly in Jira, outside the pipeline?**
A: Run `/run-pipeline phase:sync` to pull those tickets back into the docs.

**Q: Does the pipeline assign story points?**
A: No. Estimation is done by humans during sprint planning. The pipeline produces stories with enough detail for the team to estimate.

**Q: What happens to old/resolved gaps and completed stories?**
A: The orchestrator monitors document health. When resolved items exceed 50% of a file, they're archived to `docs/archive/` automatically, keeping the main files lean.

**Q: What if the pipeline crashes during Jira creation?**
A: Jira creation is resumable. The state file tracks each created ticket incrementally (immediately after creation, not batched). Re-running will skip already-created items.

**Q: How does the pipeline prevent ID collisions across focused runs?**
A: IDs are managed via `docs/.pipeline-state.json`. The orchestrator reads the registry and passes the next available ID range to each agent.

**Q: What's the difference between lite, full, and thorough modes?**
A: They control how many review gates run. Lite is for simple projects (minimal ceremony). Full is the default (standard gates + adversarial review). Thorough adds party mode, readiness gate, edge case hunting, and progressive checkpoints. See the Mode Comparison Matrix above.

**Q: Can I change the scale-adaptive thresholds?**
A: Yes. The thresholds in `scale-thresholds.md` are initial defaults that need calibration over time based on your team's experience. The orchestrator always lets you override the recommended mode.

**Q: What are the agent personas and do they matter?**
A: Each agent has a named persona (e.g., Sophia the paranoid strategist, Rex the cynical reviewer) that biases the LLM's output toward desired behavior. A "paranoid" gap analyst finds more gaps than a neutral one. This is prompt engineering, not cosmetic — it measurably affects output quality.

---

## Future Considerations (Deferred)

These enhancements are noted for future iterations:

- **Cross-Model Review**: Using different LLM providers (e.g., Claude + GPT via LiteLLM proxy) for adversarial review to get genuinely diverse perspectives from different model architectures
- **Module/Plugin System**: Domain-specific pipeline extensions (compliance module, security module, data module) that add specialized gap types, validation rules, and story template fields
- **Analysis Phase (Phase -1)**: Optional upstream capabilities (brainstorming, PRFAQ challenges, product brief creation) for when project docs don't exist yet
- **Multi-IDE Support**: Abstracting the `.claude/` structure for Cursor, Windsurf, and other AI IDEs
