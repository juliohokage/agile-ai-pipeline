# User Story Template

## Format

```markdown
### US-{epic_number}.{story_number}: {Short Title}

**As a** {role}
**I want** {goal/action}
**So that** {benefit/value}

**Priority**: {Must Have | Should Have | Could Have | Won't Have}
**Epic**: {Epic Name}

#### Acceptance Criteria

```gherkin
Given {precondition}
When {action}
Then {expected result}

Given {precondition}
When {action}
Then {expected result}
```

#### Dependencies
- {List any blocking stories or external dependencies}

#### Open Questions
- {Any unresolved questions that need stakeholder input}

#### Technical Notes
- {Implementation hints, constraints, or non-functional requirements}
```

## INVEST Checklist

Each story MUST satisfy:

- **I**ndependent: Can be developed and delivered without depending on other stories
- **N**egotiable: Details can be discussed and refined
- **V**aluable: Delivers clear value to the user or business
- **E**stimable: Contains enough detail for the team to estimate during sprint planning
- **S**mall: Completable within one sprint
- **T**estable: Has clear, verifiable acceptance criteria
