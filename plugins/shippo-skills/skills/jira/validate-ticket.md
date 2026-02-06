# Validating Ticket Completion

## Input

Jira ticket ID (e.g., `CET-1638`) or full URL.

## Validation Flow

### Step 1: Gather Ticket Information

1. Fetch ticket via `mcp__atlassian__getJiraIssue` with expand=comments,changelog
   - Extract: summary, description, acceptance criteria, status, assignee, reporter
   - Note the issue type
   - Verify Engineering Work Taxonomy is set per [reference/taxonomy.md](reference/taxonomy.md)
   - If output exceeds token limits, use `jq` to extract fields:
     ```bash
     jq '{summary: .fields.summary, status: .fields.status.name, assignee: .fields.assignee.displayName, description: .fields.description, issuetype: .fields.issuetype.name}' <file>
     ```

2. Identify linked resources from the ticket:
   - Linked Jira issues (related work, blockers, clones)
   - Confluence page links in description or comments
   - GitHub PR links in comments
   - Slack thread links in comments

### Step 2: Collect Evidence

#### From Comments (Primary Source)
- Resolution notes, status updates, documentation links
- Who worked on it and what they reported as complete
- Blockers, constraints, or follow-up items

#### From Linked Confluence Pages
- Fetch via `mcp__atlassian__getConfluencePage` if documentation was a deliverable
- Verify content exists and matches expected deliverables

#### From Linked PRs
- Search via `mcp__github-prs__search_pull_requests` with `minimal_output: true`
- Check PR status with `mcp__github-prs__get_pull_request` (`minimal_output: true`)
- Check changed files with `mcp__github-prs__get_pull_request_files`
- Verify PRs are merged if code changes were required

#### From Codebase
- If ticket involves code changes, verify implementation exists
- Use Glob/Grep to find relevant files

#### From New Relic / Observability
- If AC requires telemetry verification, ask the user:
  > "This ticket's AC references observability data. Do you have a specific NRQL query, dashboard name, or dashboard URL I should check?"
- Use a Task subagent with `mcp__new-relic__run_nrql_query` to execute queries
- Start with shorter time ranges (1 hour, then expand) to avoid timeouts
- If data lives outside New Relic (Databricks, S3), note the limitation

#### From Linked Jira Issues
- Check status of related tickets (dependencies resolved?)
- Look for follow-up tickets tracking remaining work

### Step 3: Map Evidence to Acceptance Criteria

| AC | Evidence | Status |
|----|----------|--------|
| [First criterion] | [Specific evidence found] | Met / Not Met / Partial |
| [Second criterion] | [Specific evidence found] | Met / Not Met / Partial |

### Step 4: Determine Outcome

**Can Validate (all AC met):**
- All acceptance criteria have clear evidence
- No open blockers or unresolved dependencies
- PRs merged (if applicable)
- Documentation complete (if applicable)

**Cannot Validate (needs clarification):**
- One or more AC lacks evidence
- PRs still open or not found
- Unclear if work was completed
- Conflicting information in comments

**Partial Validation:**
- Most AC met, but follow-up work tracked in separate tickets
- Constraints (e.g., code freeze) prevent full completion but mitigation documented

### Step 5: Generate Output

#### If Can Validate:
```markdown
**Validation Complete**

All acceptance criteria met:

| AC | Evidence |
|----|----------|
| [AC text] | [specific evidence with links] |

[Optional: Note any follow-up tickets or constraints]

Moving to Done.
```

#### If Cannot Validate:
```markdown
**Validation Blocked**

Unable to validate the following acceptance criteria:

| AC | Issue |
|----|-------|
| [AC text] | [what evidence is missing] |

@[assignee] Could you provide validation steps or clarify how these criteria were met?
```

#### If Partial Validation:
```markdown
**Validation Complete (Partial)**

Acceptance criteria status:

| AC | Evidence | Status |
|----|----------|--------|
| [AC text] | [evidence] | Met |
| [AC text] | [evidence/constraint] | Tracked in [TICKET-ID] |

[Explanation of why partial is acceptable]

Moving to Done.
```

## Evidence Source Priority

1. **Comments** — most reliable, contains assignee's own documentation
2. **Linked Confluence pages** — official documentation deliverables
3. **Merged PRs** — proof of code changes
4. **Linked Jira tickets** — follow-up work tracking
5. **Codebase exploration** — verify implementation exists
6. **Slack links** — team discussions (context only)

## Special Cases

| Case | Primary Evidence | Secondary Evidence |
|------|-----------------|-------------------|
| Documentation-only | Confluence page exists with expected content | Comments confirming review/sharing |
| Code changes | PR merged to main/master | Code exists, tests pass |
| Investigation/spike | Findings documented (Confluence, comments, doc) | Recommendations captured |
| Bug fix | PR merged with fix | Bug no longer reproduces |

## Notes

- Check if assignee account is active — inactive accounts cannot respond to pings
- Look for automation comments (Jira Automation) with useful status info
- Consider code freeze or other organizational constraints in comments
- When PRs are involved, check both PR status AND correct target branch
