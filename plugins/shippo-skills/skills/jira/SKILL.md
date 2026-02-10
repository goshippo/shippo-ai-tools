---
name: jira
description: Creates Jira tickets, writes ticket comments, and validates ticket completion against acceptance criteria. Applies standardized templates, engineering work taxonomy, and Shippo writing standards to all Jira operations.
allowed-tools:
  - ToolSearch(mcp__atlassian__*, mcp__github-prs__*, mcp__new-relic__*)
  - Read
  - Grep
  - Glob
  - Bash(jq)
  - Task
---

# Jira Operations

## Quick Start: Creating a Ticket

Before creating any ticket, check for duplicates:

```
mcp__atlassian__searchJiraIssuesUsingJql
  jql: "project = <KEY> AND summary ~ 'keywords' AND created >= -90d"
```

If duplicates found, reference the existing ticket instead.

Then select issue type:

| Type | When to Use | Parent |
|------|-------------|--------|
| Initiative | Quarterly strategic objective | None |
| Epic | Key Result under Initiative | Initiative |
| Story | User-facing value, feature delivery | Epic |
| Task | Internal/technical/operational work | Epic (optional) |
| Sub-task | Breakdown of parent Task/Story | Task or Story |

For the full creation workflow, templates, and taxonomy guidance, see [create-ticket.md](create-ticket.md).

## Operations

| Operation | Guide |
|-----------|-------|
| Create ticket | [create-ticket.md](create-ticket.md) — full workflow, title format, AC rules, checklist |
| Write comment | [comments.md](comments.md) — structure, examples, draft-before-posting rule |
| Validate completion | [validate-ticket.md](validate-ticket.md) — evidence gathering, AC mapping, output formats |

## Templates

Ticket body templates by issue type — load only when creating a ticket:

- [templates/initiative-epic.md](templates/initiative-epic.md) — Initiatives and Epics (Key Results)
- [templates/task.md](templates/task.md) — Tasks (internal/technical work)
- [templates/story.md](templates/story.md) — Stories (user-facing features)
- [templates/incident-follow-up.md](templates/incident-follow-up.md) — Post-incident remediation

## Reference

- [reference/taxonomy.md](reference/taxonomy.md) — Engineering Work Taxonomy (customfield_11173) values, defaults by issue type, override scenarios
- [reference/field-mapping.md](reference/field-mapping.md) — Custom field IDs, required fields, label conventions

## Writing Rules

All Jira content follows the plugin's rules (loaded separately, not repeated here):
- `writing_standards.md` — avoid blacklisted words
- `communication_style.md` — direct tone, no praise, no conversation leak

## MCP Tools

| Tool | Purpose |
|------|---------|
| `mcp__atlassian__searchJiraIssuesUsingJql` | Search tickets (duplication check, lookups) |
| `mcp__atlassian__getJiraIssue` | Fetch ticket details (use expand=comments,changelog) |
| `mcp__atlassian__createJiraIssue` | Create new ticket |
| `mcp__atlassian__editJiraIssue` | Update ticket fields |
| `mcp__atlassian__addCommentToJiraIssue` | Post comment to ticket |
| `mcp__atlassian__transitionJiraIssue` | Change ticket status |
| `mcp__atlassian__getVisibleJiraProjects` | List accessible projects |
| `mcp__atlassian__getJiraProjectIssueTypesMetadata` | Get valid issue types for a project |
| `mcp__atlassian__getJiraIssueTypeMetaWithFields` | Get field metadata (taxonomy allowed values) |
| `mcp__atlassian__lookupJiraAccountId` | Find user account IDs for assignment |

## Large API Responses

If `getJiraIssue` output exceeds token limits, the result is saved to a file. Extract fields with `jq`:

```bash
jq '{summary: .fields.summary, status: .fields.status.name, description: .fields.description}' <file>
```
