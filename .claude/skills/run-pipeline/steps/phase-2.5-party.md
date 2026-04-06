# Phase 2.5: Party Mode Review

## When This Runs
**Thorough mode only.** Skip in lite and full modes.

## Agent
Multi-agent discussion — no single agent. Uses `superpowers:dispatching-parallel-agents`
to spawn a multi-agent debate.

**Participants:**
- **Sophia** (Gap Analyst) — defends and expands the gap analysis
- **Marcus** (Epic Decomposer) — challenges from an architecture/scope perspective
- **Rex** (Adversarial Reviewer) — pokes holes in everything

## Input
- `docs/GAP_ANALYSIS.md` (just reviewed and approved by human at Phase 2 checkpoint)
- `docs/PROJECT_CONTEXT.md` (for reference)

## Process

### Step 1: Set up the discussion

Use `superpowers:dispatching-parallel-agents` to launch 3 agents in a structured debate format.

Frame the discussion prompt for each agent:

**Sophia (Gap Analyst):**
"You are Sophia. Review docs/GAP_ANALYSIS.md that you produced. Defend your gap analysis
but also look for gaps you missed. Challenge yourself: what domains did you under-investigate?
What assumptions did you make that should be explicit? Present your findings as additions."

**Marcus (Epic Decomposer):**
"You are Marcus. Review docs/GAP_ANALYSIS.md from an architecture perspective.
Are any gaps actually the same gap viewed from different angles? Are any gaps too broad
to be actionable? Are there structural/architectural gaps that a functional analyst
would miss? Present your findings as challenges or additions."

**Rex (Adversarial Reviewer):**
"You are Rex. Review docs/GAP_ANALYSIS.md with maximum cynicism. What's missing?
What risks are underrated? What gaps exist between the gaps? Where would this project
fail in production that nobody has considered? Present your findings as challenges."

### Step 2: Collect and synthesize findings

After all three agents complete:
1. Collect their findings
2. De-duplicate: if multiple agents flagged the same gap, merge into one finding
3. Categorize new findings: New Gap, Gap Enhancement, Challenge to Existing Gap

### Step 3: Append to GAP_ANALYSIS.md

Append a new section to `docs/GAP_ANALYSIS.md`:

```markdown
### Multi-Agent Review (Added: {date})

**Participants**: Sophia (Gap Analyst), Marcus (Epic Decomposer), Rex (Adversarial Reviewer)

#### New Gaps Identified
{new gaps in standard gap format, continuing ID sequence}

#### Enhancements to Existing Gaps
{existing gap ID → what was added/changed}

#### Challenges Raised
{challenges that don't create new gaps but require stakeholder attention}
```

### Step 4: Update state

Update `docs/.pipeline-state.json`:
- `ids.highest_gap` → updated if new gaps were added

## Output
- Updated `docs/GAP_ANALYSIS.md` with multi-agent review findings appended

## State Updates
```json
{
  "ids": { "highest_gap": {n} }
}
```

## Checkpoint
No additional human checkpoint — Phase 2 checkpoint already covered the initial gap analysis.
The multi-agent additions are visible in the next checkpoint (Phase 3) where the user reviews
the Epic decomposition based on the complete gap analysis.
