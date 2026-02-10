# Example: Creating a Task with Incident Follow-up

## Scenario

User: "Create a ticket for adding connection pooling to shippo_py3 NSQ consumers. This is follow-up from INC-2892."

## Steps

### 1. Duplication Check

```
mcp__atlassian__searchJiraIssuesUsingJql
  jql: "project = CET AND summary ~ 'connection pooling NSQ' AND created >= -90d"
```

Result: No duplicates found.

### 2. Issue Type Selection

Internal/technical work → **Task**
Post-incident context → override taxonomy to **Incident Follow-up**

### 3. Apply Template (incident-follow-up.md)

**Title:** `[Q1/2026][SRE] Add connection pooling to shippo_py3 NSQ consumers`

**Description:**
Follow-up from [INC-2892](link). Add connection pooling to prevent NSQ connection exhaustion during traffic spikes.

**Context:**
* Root cause: Each message created new DB connection without pooling
* Impact: 503 errors during peak load, 15 min downtime

**AC:**
* Connection pooling implemented across all NSQ consumers
* Load test confirms no connection exhaustion under 2x peak traffic

### 4. Field Values

| Field | Value |
|-------|-------|
| Issue Type | Task |
| Taxonomy | Incident Follow-up (ID: 12012) |
| Labels | `incident-follow-up`, `ready-for-grooming` |
| Parent | (linked to relevant Epic if applicable) |
