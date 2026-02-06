# Validate Ticket Command

## Usage
When user provides a Jira ticket ID or URL, perform validation assessment to determine if the ticket can be moved to Done.

## Input
- Jira ticket ID (e.g., `CET-1638`) or full URL (e.g., `https://shippo.atlassian.net/browse/CET-1638`)

## Validation Flow

### Step 1: Gather Ticket Information

1. **Fetch ticket details** using `mcp__atlassian__getJiraIssue` with expand=comments,changelog
   - Extract: summary, description, acceptance criteria, status, assignee, reporter
   - Note the issue type (Story, Task, Bug, etc.)
   - Verify Engineering Work Taxonomy is set per @.claude/rules/engineering_work_taxonomy.md
   - If the result exceeds token limits, the output will be saved to a file. Use Bash with `jq` to extract structured fields:
     ```
     jq '{summary: .fields.summary, status: .fields.status.name, assignee: .fields.assignee.displayName, description: .fields.description, issuetype: .fields.issuetype.name}' <file>
     ```
     Then extract comments and links separately with targeted jq queries.

2. **Identify linked resources** from the ticket:
   - Linked Jira issues (related work, blockers, clones)
   - Confluence page links in description or comments
   - GitHub PR links in comments
   - Slack thread links in comments

### Step 2: Collect Evidence from All Sources

#### From Comments (Primary Source)
- Look for resolution notes, status updates, and documentation links
- Identify who worked on it and what they reported as complete
- Note any blockers, constraints, or follow-up items mentioned

#### From Linked Confluence Pages
- Fetch using `mcp__atlassian__getConfluencePage` if documentation was a deliverable
- Verify the content exists and matches expected deliverables

#### From Linked PRs (if applicable)
- Search for PRs using `mcp_github-prs_search_pull_requests` with `minimal_output: true` to avoid oversized results
- Use `mcp_github-prs_get_pull_request` with `minimal_output: true` to check PR status (merged/open/closed)
- Use `mcp_github-prs_get_pull_request_files` to see what was changed
- Verify PRs are merged if code changes were required

#### From Codebase (if applicable)
- If the ticket involves code changes, verify the implementation exists
- Use Glob/Grep to find relevant files mentioned in the ticket
- Confirm expected functionality is present

#### From New Relic / Observability (if applicable)
- If AC requires telemetry verification (metrics, logs, dashboards), ask the user:
  > "This ticket's AC references observability data. Do you have a specific NRQL query, dashboard name, or dashboard URL I should check?"
- Use a Task subagent (subagent_type: general-purpose) with `mcp__new-relic__run_nrql_query` to execute queries
  - Start with shorter time ranges (1 hour, then expand) to avoid timeouts
  - If a query times out, reduce the time window or add LIMIT/filters before retrying
- If user provides a dashboard name, use `mcp__new-relic__list_dashboards` to find it, then `mcp__new-relic__get_dashboards` to inspect widgets and queries
- If data lives outside New Relic (e.g., Databricks, S3), note the limitation and document what alternative evidence was found

#### From Linked Jira Issues
- Check status of related tickets (are dependencies resolved?)
- Look for follow-up tickets that track remaining work

### Step 3: Map Evidence to Acceptance Criteria

Create a mapping table:

| AC | Evidence | Status |
|----|----------|--------|
| [First acceptance criterion] | [Specific evidence found] | Met/Not Met/Partial |
| [Second acceptance criterion] | [Specific evidence found] | Met/Not Met/Partial |

### Step 4: Determine Validation Outcome

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
- Most AC met, but follow-up work properly tracked in separate tickets
- Constraints (e.g., code freeze) prevent full completion but mitigation documented

### Step 5: Generate Output

#### If Can Validate:
Generate validation comment in this format:

```markdown
**Validation Complete**

All acceptance criteria met:

| AC | Evidence |
|----|----------|
| [AC text] | [specific evidence with links] |
| [AC text] | [specific evidence with links] |

[Optional: Note any follow-up tickets or constraints]

Moving to Done.
```

#### If Cannot Validate:
Generate request for validation steps:

```markdown
**Validation Blocked**

Unable to validate the following acceptance criteria:

| AC | Issue |
|----|-------|
| [AC text] | [what evidence is missing] |

@[assignee] Could you provide validation steps or clarify how these criteria were met?
```

#### If Partial Validation:
Generate conditional validation comment:

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

1. **Comments** - Most reliable, contains assignee's own documentation
2. **Linked Confluence pages** - Official documentation deliverables
3. **Merged PRs** - Proof of code changes
4. **Linked Jira tickets** - Follow-up work tracking
5. **Codebase exploration** - Verify implementation exists
6. **Slack links** - Team discussions and decisions (context only)

## Special Cases

### Documentation-Only Tickets
- Primary evidence: Confluence page exists with expected content
- Secondary: Comments confirming documentation was reviewed/shared

### Code Change Tickets
- Primary evidence: PR merged to main/master
- Secondary: Code exists in codebase, tests pass

### Investigation/Spike Tickets
- Primary evidence: Findings documented (Confluence, comments, or linked doc)
- Secondary: Recommendations or next steps captured

### Bug Fix Tickets
- Primary evidence: PR merged with fix
- Secondary: Verification that bug no longer reproduces (if testable)

## Notes
- Always check if assignee account is active - inactive accounts cannot respond to pings
- Look for automation comments (Jira Automation) that may contain useful status info
- Consider code freeze or other organizational constraints mentioned in comments
- When PRs are involved, check both the PR status AND if it's merged to the correct branch
