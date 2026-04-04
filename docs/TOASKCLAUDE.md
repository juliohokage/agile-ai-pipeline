1. Remove T-Shirt Size Estimation from Story Writer

  Current state: The story-writer agent at .claude/agents/story-writer.md:46 uses **Estimate**: {S | M | L | XL} for stories. The template at docs/templates/story-template.md:12 also includes **Estimate**: {S | M | L |
  XL}. The validator checks "Estimable" as an INVEST criterion. The user-story skill at .claude/skills/user-story/SKILL.md:49 also estimates size.

  What to change: Remove Estimate lines from story-writer, story-template, and user-story skill. Keep the "Estimable" INVEST check as-is (it validates the story has enough info for a team to estimate, which is different
   from the agent providing the estimate). The validator should stop checking the estimate field exists, but keep validating that the story is estimable (enough detail).

  I'll make these changes. Shall I proceed?

  ---
  2. Remove Confluence from Doc Ingester + Template Questions

  Current state in doc-ingester.md:85:
  ▎ If Confluence MCP is available, search for relevant project spaces

  This is the only reference to Confluence in the doc-ingester. It's a single line in Step 1.

  Confluence templates (docs/templates/confluence/*.xml): These 5 templates (user-story.xml, epic.xml, spike.xml, sprint-retro.xml, decision-log.xml) are Confluence Storage Format markup. Their purpose was to
  export/publish pipeline output TO Confluence pages — they are rendering templates for displaying stories/epics/spikes/retros/decisions in Confluence wiki format. No agent currently references or uses them. They were
  created as future capability for publishing to Confluence but are not wired into any agent or skill.

  Verdict: Since this is a GitOps project with mkdocs, these Confluence XML templates are dead code. Safe to delete the entire docs/templates/confluence/ directory.

  The CONTEXT.MD / PROJECT_CONTEXT.md Contradiction Question

  You are correct — there IS a contradiction, but it's intentional design with poor documentation.

  Here's the factual breakdown:

  - Doc-ingester.md:26-27: Mode 1 (Full Ingestion) is triggered "when docs/PROJECT_CONTEXT.md does not exist." It creates the file.
  - run-pipeline SKILL.md:72: Phase prerequisite table says analyze requires docs/PROJECT_CONTEXT.md exists, decompose requires it, etc.
  - run-pipeline SKILL.md:88-99: Phase 1 (Ingest) runs FIRST in full discovery — it creates PROJECT_CONTEXT.md. The prerequisite for ingest is only that docs/ has at least one file.

  So the flow is:
  1. Phase 1 (ingest) → prerequisite: docs/ has files → creates PROJECT_CONTEXT.md
  2. Phase 2 (analyze) → prerequisite: PROJECT_CONTEXT.md exists (created by Phase 1)

  The pipeline does NOT stop if PROJECT_CONTEXT.md doesn't exist at the start. It only requires it before Phase 2. Phase 1 creates it. The prerequisite checks in the table (line 67-78) are per-phase, not global. If you
  run /run-pipeline (full discovery), Phase 1 runs first and creates it.

  Where the confusion arises: If you run /run-pipeline phase:analyze directly (skipping ingest), it WILL block because PROJECT_CONTEXT.md doesn't exist. That's correct behavior — it tells you to run ingest first. The
  Mode Detection table (line 42-47) also says: if PROJECT_CONTEXT.md doesn't exist → Discovery mode (which starts from Phase 1).

  You were wrong about the contradiction — the design is actually correct, but the documentation makes it confusing because the prerequisite table is listed before the phase descriptions and reads like a global
  prerequisite check.

  ---
  3. Token Optimization

  Current problem: Every agent starts with "Read docs/PROJECT_CONTEXT.md." In a full pipeline run, this file is read by:
  1. Doc Ingester (creates it)
  2. Gap Analyst (reads it)
  3. Epic Decomposer (reads it)
  4. Story Writer × N epics (each reads it)
  5. Story Validator (doesn't read it, reads stories)
  6. Jira Sync (doesn't need it)

  With 5 Epics, that's 7 reads of the same file. If PROJECT_CONTEXT.md is 3000 tokens, that's ~21K tokens wasted on re-reads.

  Approaches to optimize:

  Approach A: Context Injection by Orchestrator (Recommended)

  The orchestrator already dispatches subagents via the Agent tool. Instead of each agent reading PROJECT_CONTEXT.md themselves, the orchestrator reads it once and injects the relevant sections into each subagent's
  prompt. For example:
  - Gap Analyst needs: full context
  - Epic Decomposer needs: architecture + requirements sections only
  - Story Writer needs: only the sections relevant to its Epic's domain

  Pros: Massive token savings, agents get only what they need.
  Cons: Orchestrator becomes more complex. Requires maintaining which sections each agent needs.

  Approach B: Sectioned Context Files

  Split PROJECT_CONTEXT.md into sections (architecture.md, requirements.md, stakeholders.md, etc.) and have each agent read only what it needs.

  Pros: Simple to implement, each agent declares its inputs.
  Cons: Breaks the "single source of truth" pattern. More files to manage.

  Approach C: Summary + Detail Pattern

  Create a short PROJECT_CONTEXT_SUMMARY.md (500 tokens max) for agents that need light context (Story Writer), while keeping the full file for deep analysis agents (Gap Analyst).

  Pros: Simple, backward-compatible. Minimal changes.
  Cons: Summary may miss details.

  My recommendation: Approach A for the best ROI. The orchestrator is already the hub — adding context injection is natural and gives you the most control.

  ---
  4. Maintainability & Human Workflow

  Archiving Done/Cancelled Epics and Stories

  Current state: There is no archival mechanism. Files accumulate indefinitely. The jira-sync agent adds status badges ([DONE ✅]) but never moves files.

  Options:

  1. Jira Sync Agent handles it (recommended): After sync, when all stories in an Epic are DONE, the agent could move docs/stories/EPIC-{n}-stories.md to docs/archive/stories/ and mark the Epic as ✅ Complete in
  EPICS.md. This is automatable and consistent.
  2. Manual archive folder: Human creates docs/archive/ and moves files. Simple but error-prone and requires discipline.
  3. Keep everything, filter by status: Don't move anything — just add status badges. The validator and other agents already filter. This is the GitOps-purist approach (git history IS the archive).

  My recommendation: Option 3 (keep + status badges) for simplicity, with an optional /run-pipeline phase:archive command that moves completed Epics to docs/archive/ when the human decides to clean up.

  Meeting Transcripts

  Great idea. Add docs/meeting-transcripts/ as an ingestion source. The doc-ingester already scans docs/ recursively and skips docs/stories/ and docs/templates/. Meeting transcripts would be automatically picked up.

  Suggested structure: docs/meeting-transcripts/{YYYY-MM-DD}-{topic}.md

  The doc-ingester's manifest will track them and only re-ingest when modified. They'd feed into PROJECT_CONTEXT.md's "Key Decisions" and "Open Items" sections, which then flow into gap analysis and epic planning.

  No code change needed for ingestion — just create the directory and add files. The doc-ingester already handles arbitrary .md files in docs/.

  ---
  5. Testing Without Jira

  Yes, this is feasible with the current state.

  The pipeline has human checkpoints after Phase 2, 3, and 5. At Phase 5's checkpoint (line 175 of SKILL.md), it asks:

  ▎ "Reply 'create-jira' to push to Jira, or provide feedback."

  You can simply NOT say 'create-jira'. Say "skip" or "stop here" and the pipeline stops. Phases 1-5 are entirely local — no Jira MCP needed.

  Additionally, the Jira safety gate hook in settings.json only blocks Jira tool calls. It doesn't affect Phases 1-5 at all. Even if the MCP server isn't configured, Phases 1-5 will run fine because they only read/write
   local markdown files.

  One issue: If you run /run-pipeline without modification, the orchestrator will try to proceed to Phase 6 after Phase 5 approval. You need to explicitly tell it to stop. The orchestrator's wording ("Reply
  'create-jira' to push to Jira") already gives you an opt-out. You're safe.

  ---
  6. Full Critical Review