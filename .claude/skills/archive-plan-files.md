---
name: archive-plan-files
description: Archive plan directories for closed Jira tickets
---

# Archive Plan Files

Archives plan directories in `.claude/plans/` when their associated Jira tickets are closed.

## When to Use

- Periodic cleanup of completed work
- Before sprint planning to clear clutter
- When plans directory has many old ticket directories

## Procedure

### 1. Identify Ticket Directories

List directories in `.claude/plans/` that match ticket patterns (e.g., `CET-1234`, `PROJ-567`):

```bash
ls .claude/plans/ | grep -E '^[A-Z]+-[0-9]+$'
```

### 2. Query Jira Status

**REQUIRED** - Always query Jira for live ticket status. Never infer status from local files, file age, or directory contents.

For each ticket directory found, query Jira:

```
Use mcp__atlassian__getJiraIssue with:
- cloudId: bcd1ff35-af4b-43de-88a4-a2bc11d9f807
- issueIdOrKey: <ticket-key>
- fields: ["status", "summary"]
```

Query all tickets in parallel for efficiency.

**Fallback**: If the Atlassian MCP tool is unavailable, use the Bash tool:
```bash
curl -s -u "$ATLASSIAN_EMAIL:$ATLASSIAN_API_TOKEN" \
  "https://shippo.atlassian.net/rest/api/3/issue/<ticket-key>?fields=status,summary"
```

### 3. Categorize by Status

Group tickets into:
- **Archive**: Status category is "Done" (statusCategory.key === "done")
- **Keep**: All other statuses (To Do, In Progress, Waiting Deployment, etc.)

### 4. Present Summary

Show a table before taking action:

| Ticket | Summary | Status | Action |
|--------|---------|--------|--------|
| CET-1234 | Fix bug | Done | Archive |
| CET-5678 | New feature | In Progress | Keep |

**STOP**: Wait for user approval before moving any files.

### 5. Move Closed Tickets

Create archive directory and move closed ticket directories:

```bash
mkdir -p .claude/plans/archive
mv .claude/plans/<TICKET-1> .claude/plans/<TICKET-2> ... .claude/plans/archive/
```

### 6. Verify Results

List both directories to confirm:

```bash
echo "=== Archived ===" && ls .claude/plans/archive/
echo "=== Remaining ===" && ls .claude/plans/
```

## Notes

- Only moves directories matching ticket patterns (PROJECT-NUMBER format)
- Non-ticket directories (e.g., `deployment-confidence`, `sprint-25`) are ignored
- Does not delete anything - just moves to archive subdirectory
- Archive can be manually cleaned up later if needed
