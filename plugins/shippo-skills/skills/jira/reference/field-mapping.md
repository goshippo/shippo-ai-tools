# Jira Field Mapping

## Required Fields

| Field | API Field | Value |
|-------|-----------|-------|
| Issue Type | `issuetype.id` | Initiative / Epic / Task / Story / Sub-task |
| Summary | `summary` | Title following `[Q#/YYYY][Category][KR#] [Name]` format |
| Parent | `parent.key` | Link to parent Epic/Initiative (Sub-tasks link to Task/Story) |
| Engineering Work Taxonomy | `customfield_11173.id` | See [taxonomy.md](taxonomy.md) for values |
| Labels | `labels` | See label defaults below |

## Custom Fields

| Field | Custom Field ID | Purpose |
|-------|----------------|---------|
| Engineering Work Taxonomy | `customfield_11173` | Work type categorization for reporting |

## Label Defaults

| Context | Labels |
|---------|--------|
| Standard ticket | `ready-for-grooming` |
| Incident follow-up | `incident-follow-up` |
| Both contexts apply | Both labels |

## Shippo Cloud ID

```
bcd1ff35-af4b-43de-88a4-a2bc11d9f807
```

Used in all `mcp__atlassian__` tool calls requiring `cloudId`.

## Example: Create Ticket API Call

```
mcp__atlassian__createJiraIssue
  cloudId: "bcd1ff35-af4b-43de-88a4-a2bc11d9f807"
  projectIdOrKey: "CET"
  fields: {
    "summary": "[Q1/2026][AI][KR1] Example ticket title",
    "issuetype": {"id": "<issue_type_id>"},
    "customfield_11173": {"id": "11396"},
    "labels": ["ready-for-grooming"]
  }
```
