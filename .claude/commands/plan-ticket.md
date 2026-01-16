---
name: plan-ticket
description: Deep-dive Jira ticket analysis and implementation planning
---

# Ticket Researcher Command

## Usage
When user provides a Jira ticket ID or URL, analyze it deeply and create an implementation plan.

## Steps
1. **Fetch ticket details** using `mcp_atlassian_getJiraIssue` with the ticket ID/key
2. **Get comments** using `mcp_atlassian_searchJiraIssuesUsingJql` to find related discussions
3. **Generate 100-word summary** covering:
   - What the ticket is asking for
   - Current status and priority
   - Key stakeholders/assignees
4. **Create 100-word high-level plan** with:
   - Major implementation steps
   - Technical approach
   - Dependencies or blockers
5. **Ask 3 highest priority clarifying questions** such as:
   - Scope boundaries
   - Technical constraints
   - Acceptance criteria gaps
6. **Wait for developer agreement** on the plan
7. **Generate 100-line markdown file** named `CET-{id}-plan.md` with:
   - Full ticket context
   - Detailed implementation steps
   - Technical considerations
   - Acceptance criteria checklist
