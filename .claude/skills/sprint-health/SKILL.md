---
name: sprint-health
description: >
  Check sprint health by querying Jira for current sprint data.
  Shows completion rate, in-progress work, blockers, and overall sprint status.
  Use between sprint syncs for a quick health check.
arguments:
  - name: project-key
    description: >
      Jira project key. Defaults to ADDPROJECTKEY.
    default: ADDPROJECTKEY
  - name: sprint
    description: >
      Optional. Sprint name or "active" (default) for current sprint.
    default: active
---

# Sprint Health Check

Quick sprint health assessment by querying Jira directly.
Does NOT modify any local docs — read-only check.

## Process

### Step 1: Query Jira for sprint data

```
Call: jira_search
  jql: "project = {project-key} AND sprint in openSprints() ORDER BY priority DESC"
```

If a specific sprint name is provided:
```
Call: jira_search
  jql: "project = {project-key} AND sprint = '{sprint}' ORDER BY priority DESC"
```

### Step 2: Classify issues

Group all returned issues:

| Status Category | Statuses |
|----------------|----------|
| **Done** | Done, Closed, Resolved |
| **In Progress** | In Progress, In Review, In QA |
| **To Do** | To Do, Open, Backlog |
| **Blocked** | Any issue with "Blocked" flag or "Impediment" label |

### Step 3: Calculate health metrics

```
Completion Rate    = Done / Total * 100
In Progress Rate   = In Progress / Total * 100
Not Started Rate   = To Do / Total * 100
Blocker Count      = issues with blocked status or impediment label
```

### Step 4: Present sprint health dashboard

```markdown
## Sprint Health Dashboard

**Sprint**: {sprint name}
**Project**: {project-key}
**Checked**: {date}

### Progress

| Status | Count | % |
|--------|-------|---|
| Done | {n} | {%} |
| In Progress | {n} | {%} |
| To Do | {n} | {%} |
| **Total** | **{n}** | **100%** |

### Health Indicator

{HEALTHY | AT RISK | UNHEALTHY}

- HEALTHY: >60% done OR >80% (done + in progress) with no blockers
- AT RISK: 40-60% done OR blockers present
- UNHEALTHY: <40% done AND significant work still in To Do

### Blockers

| Issue | Summary | Assignee | Days Blocked |
|-------|---------|----------|-------------|
| {key} | {summary} | {assignee} | {days} |

### By Priority

| Priority | Done | In Progress | To Do |
|----------|------|-------------|-------|
| Highest | {n} | {n} | {n} |
| High | {n} | {n} | {n} |
| Medium | {n} | {n} | {n} |

### Items Completed

| Issue | Summary | Type |
|-------|---------|------|
| {key} | {summary} | {type} |

### Items At Risk (In Progress, not moving)

| Issue | Summary | Assignee | Days in Status |
|-------|---------|----------|---------------|
| {key} | {summary} | {assignee} | {days} |
```

### Step 5: Recommendations

Based on the data, suggest actions:
- If blockers exist: "Resolve {n} blockers — these are preventing sprint completion"
- If too much in To Do: "Consider descoping — {n} items haven't started"
- If healthy: "Sprint is on track. {n}/{total} items completed."

## Rules

- Read-only — never modify Jira issues
- Never modify local docs — this is a quick check only
- If Jira MCP is not available, tell the user and suggest running `/run-pipeline phase:sync` instead
- If no active sprint exists, report that and suggest checking the board
