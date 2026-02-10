# Creating Jira Tickets

## Workflow

```
1. Search existing tickets (duplication check — required)
2. Identify issue type (Initiative / Epic / Task / Story / Sub-task / Bug)
3. Determine parent (link to Epic/Initiative if applicable)
4. Select taxonomy (see reference/taxonomy.md)
5. Apply template from templates/
6. Write succinct content (2-3 sentences max)
7. Define 1-3 measurable AC
8. Create via MCP tool
9. Return ticket key and link
```

## Duplication Check (Required)

```
mcp__atlassian__searchJiraIssuesUsingJql
  jql: "project = <KEY> AND summary ~ 'keywords' AND created >= -90d"
```

If matches found: reference existing ticket, don't create new.

## Issue Type Selection

| Type | When to Use | Parent | Template |
|------|-------------|--------|----------|
| Initiative | Quarterly strategic objective | None | [initiative-epic.md](templates/initiative-epic.md) |
| Epic | Key Result under Initiative | Initiative | [initiative-epic.md](templates/initiative-epic.md) |
| Story | User-facing value, feature delivery | Epic | [story.md](templates/story.md) |
| Task | Internal/technical/operational work | Epic (optional) | [task.md](templates/task.md) |
| Sub-task | Breakdown of parent Task/Story | Task or Story | Inherits from parent |

**Story vs Task:** Story has a user-facing outcome ("As a [user], I want..."). Task is backend ops, infrastructure, refactors — no user-facing value.

**Sub-task rule:** Sub-tasks inherit taxonomy and labels from parent. Do not override unless explicitly instructed.

## Title Format

```
[Quarter/Year][Category][KR#] [Name]
```

| Component | Format | Examples |
|-----------|--------|----------|
| Quarter/Year | `[Q1/2025]` | `[Q3/2025]`, `[Q4/2026]` |
| Category | `[AI]`, `[INFRA]`, `[DATA]` | Project-specific tags |
| KR# | `[KR1]`, `[KR2]` | Key Result reference (if applicable) |
| Name | Concise description | 5-10 words max |

Examples:
- `[Q3/2025][AI][KR4] Promote AI Adoption`
- `[Q3/2025][INFRA][KR2] Setup Artifactory publishing for MCP servers`

## Description Standards

**2-3 sentences maximum.**

| Section | When to Include |
|---------|-----------------|
| Description | Always — brief what/why |
| Scope | When boundaries matter — included/excluded |
| Technical Details | Tasks only — implementation specifics |
| Background | Only if context needed |
| Story Content | Stories only — key elements/deliverables |

## Acceptance Criteria

**1-3 criteria maximum.** If more needed, split the ticket.

Rules: measurable, outcome-focused, bullet points (not numbered), no process steps.

Good:
- Dashboard displays 95th percentile latency under 200ms
- API returns valid response within 2 seconds
- Documentation published to Confluence

Bad:
- Run the tests
- Deploy to staging
- Review with team

## Engineering Work Taxonomy

See [reference/taxonomy.md](reference/taxonomy.md) for full guidance.

Quick defaults:

| Issue Type | Default Taxonomy |
|------------|-----------------|
| Initiative | Feature - Product Enhancement |
| Epic | Inherits from Initiative context |
| Story | Feature - Product Enhancement |
| Task | Technical Investment (Tech Inv) |
| Sub-task | Inherit from parent |
| Bug | System Patch |

Override the Task default when context clearly indicates a different category (incident follow-up, telemetry investigation, carrier compliance, etc.). See [reference/taxonomy.md](reference/taxonomy.md) for override scenarios.

## Labels

| Context | Labels |
|---------|--------|
| Standard ticket | `ready-for-grooming` |
| Incident follow-up | `incident-follow-up` |
| Both apply | Include both labels |

## Incident Follow-up Tickets

1. Set taxonomy to "Incident Follow-up"
2. Add `incident-follow-up` label
3. Reference INC-xxxx IDs in description
4. Link to parent epic/story if applicable
5. Use [templates/incident-follow-up.md](templates/incident-follow-up.md)

## Pre-Creation Checklist

- [ ] Searched for duplicates using JQL
- [ ] Confirmed issue type (Story vs Task)
- [ ] Title follows `[Q#/YYYY][Category][KR#] [Name]` format
- [ ] Description is 2-3 sentences max
- [ ] AC is 1-3 measurable criteria max
- [ ] Parent epic/initiative linked (if applicable)
- [ ] Labels set per context
- [ ] Engineering Work Taxonomy set per issue type guidance
- [ ] No blacklisted words (per writing_standards.md)
- [ ] Template structure followed exactly

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No duplication check | Search JQL first |
| 5+ acceptance criteria | Split into multiple tickets |
| Description paragraph | Cut to 2-3 sentences |
| Process steps as AC | Focus on outcomes |
| Missing KR reference | Add `[KR#]` to title |
| Task for user-facing work | Use Story instead |
| Story for internal work | Use Task instead |
| No parent link | Connect to Epic/Initiative |
| Wrong taxonomy for incident | Use "Incident Follow-up" |
