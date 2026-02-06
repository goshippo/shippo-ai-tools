# Example: Validating a Ticket with PR Evidence

## Scenario

User: "Validate CET-1638"

## Steps

### 1. Fetch Ticket

```
mcp__atlassian__getJiraIssue
  issueIdOrKey: "CET-1638"
  expand: "comments,changelog"
```

Summary: "Add retry logic to carrier API calls"
AC:
1. Retry logic implemented with exponential backoff
2. Unit tests cover retry scenarios
3. Monitoring dashboard updated

### 2. Collect Evidence

**From Comments:**
- Assignee posted: "PR #245 merged with retry implementation. Tests added."
- Link to New Relic dashboard in comment

**From PR:**
```
mcp__github-prs__get_pull_request
  owner: "goshippo"
  repo: "shippo_py3"
  pullNumber: 245
  minimal_output: true
```
PR #245: merged, 4 files changed

**From PR Files:**
- `app/services/carrier_api.py` — retry logic added
- `tests/test_carrier_api.py` — 3 new test cases

### 3. Map Evidence to AC

| AC | Evidence | Status |
|----|----------|--------|
| Retry logic with exponential backoff | PR #245 merged, `carrier_api.py` changes | Met |
| Unit tests cover retry scenarios | 3 new tests in `test_carrier_api.py` | Met |
| Monitoring dashboard updated | NR dashboard link in comments, verified exists | Met |

### 4. Output

**Validation Complete**

All acceptance criteria met:

| AC | Evidence |
|----|----------|
| Retry logic with exponential backoff | PR [#245](link) merged — `carrier_api.py` |
| Unit tests cover retry scenarios | 3 test cases in `test_carrier_api.py` |
| Monitoring dashboard updated | [Dashboard link](link) confirmed |

Moving to Done.
