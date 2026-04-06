---
name: user-story
description: >
  Quick single User Story or Spike generation without running the full pipeline.
  Use when you need to draft one story for a known topic, add a story to an existing Epic,
  or create a quick Spike for a research question.
arguments:
  - name: topic
    description: >
      What the story is about. Can be a feature, a research question, or a task.
      Example: "user authentication via SSO", "evaluate Mimir vs VictoriaMetrics"
  - name: epic
    description: >
      Optional. Epic number to attach this story to. If provided, the story will be
      appended to the existing Epic story file with correct numbering.
  - name: type
    description: >
      Optional. "story" (default) or "spike" for research/study tasks.
    default: story
  - name: validate
    description: >
      Optional flag. If set, automatically validates the story after saving
      by dispatching the Story Validator in subset mode.
    default: false
  - name: jira
    description: >
      Optional flag. If set (and --validate passes), automatically creates
      a Jira ticket for the story. Requires validation to pass first.
    default: false
---

# Quick User Story Generator

Generate a single User Story or Spike without running the full pipeline.
This is for ad-hoc story creation when you already know what you want.

## Process

### Step 1: Load context

- Read `docs/PROJECT_CONTEXT.md` if it exists (for domain understanding)
- If `$ARGUMENTS.epic` is provided, read `docs/EPICS.md` and `docs/stories/EPIC-{epic}-stories.md`
  to understand the Epic context and get the next story number
- Read `docs/templates/story-template.md` for format reference

### Step 2: Determine story type

- If `$ARGUMENTS.type` is "spike" OR the topic contains words like "evaluate", "compare",
  "study", "research", "investigate", "POC", "prototype" → create a Spike
- Otherwise → create a User Story

### Step 3: Draft the story

**For User Stories:**
1. Identify the user role/persona
2. Write "As a / I want / So that"
3. Write 2-5 Gherkin acceptance criteria (Given/When/Then)
4. Assign priority (Must Have / Should Have / Could Have)
5. Run INVEST check

**For Spikes:**
1. Write "As a / I want to study-evaluate / So that we can decide"
2. Define time-box (1-5 days)
3. Write acceptance criteria (what the output deliverable must contain)
4. Define the output document path (`docs/decisions/{topic}.md`)
5. List follow-up stories that will be created after the spike

### Step 4: Present to user

Show the drafted story in full format. Ask:
"Does this look right? I can adjust the scope, acceptance criteria, or priority. Reply 'save' to write it to the story file, or provide feedback."

### Step 5: Save (if approved)

- If `$ARGUMENTS.epic` is provided:
  - Append to `docs/stories/EPIC-{epic}-stories.md` with correct numbering
  - Update the Story Summary table at the top of that file
- If no epic specified:
  - Save to `docs/stories/STANDALONE-stories.md` (for stories not yet assigned to an Epic)
  - Note: These should be assigned to an Epic during the next backlog refinement

### Step 6: Auto-validate (if --validate flag is set)

If `$ARGUMENTS.validate` is true:
1. Dispatch the `story-validator` subagent in SUBSET mode for just the saved story file
2. Report validation result: PASS / FAIL / WARNINGS
3. If FAIL: show the issues and ask if the user wants to fix them
4. If PASS: update `docs/.pipeline-state.json` validation status and stories hash

If `$ARGUMENTS.validate` is false:
- Ask: "Want me to validate this story? (Use `--validate` flag for automatic validation)"

### Step 7: Auto-create Jira ticket (if --jira flag is set)

If `$ARGUMENTS.jira` is true AND validation passed (Step 6):
1. Run pre-flight checks (same as Phase 6):
   - `docs/.pipeline-state.json` → `validation.status` == "PASS"
   - Stories hash matches
   - Project key is configured
2. Create the Jira ticket using `mcp__atlassian__jira_create_issue`
3. If epic is specified, link to the Epic's Jira ticket (if it exists)
4. Update `docs/.pipeline-state.json` → `jira_creation.created`
5. Report: "Jira ticket created: {JIRA-KEY}"

If `$ARGUMENTS.jira` is true BUT validation failed:
- "Cannot create Jira ticket — validation failed. Fix the issues first."

If `$ARGUMENTS.jira` is false:
- Ask: "Want me to create this in Jira now? (Use `--validate --jira` flags for automatic flow)"

## Rules

- The `--jira` flag requires `--validate` to also be set (or validation to have already passed)
- Stories created here with `--validate` update the validation gate in `.pipeline-state.json`
- If the topic is too large for a single story, suggest splitting and offer to run the full pipeline instead
- Always show the INVEST check results (for stories) or Spike criteria check (for spikes)
