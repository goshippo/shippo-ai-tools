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
3. **Delegated research phase** — spawn sub-agents (in parallel where possible) to protect the main context window:
   - **codebase-locator** (if available): Find relevant files, modules, entry points
   - **codebase-analyzer** (if available): Understand how code works with file:line references
   - **web-search-researcher** (if available): External docs, library APIs, migration guides
   - **Custom Task sub-agents**: For domain-specific questions — spawn with a focused prompt
   - Limit to **3-5 sub-agent dispatches**. If more needed, ask the developer.
   - Each sub-agent returns a **focused summary under 200 words**.
4. **Generate 100-word summary** covering:
   - What the ticket is asking for
   - Current status and priority
   - Key stakeholders/assignees
5. **Create 100-word high-level plan** with:
   - Major implementation steps
   - Technical approach
   - Dependencies or blockers
6. **Ask 3 highest priority clarifying questions** such as:
   - Scope boundaries
   - Technical constraints
   - Acceptance criteria gaps
7. **Wait for developer agreement** on the plan
8. **Discover available skills and plugins** — before writing the plan, scan for installed skills, commands, and plugins that could assist with implementation:
   - Check `.claude/commands/`, `.claude/skills/`, and any loaded plugin skills
   - Identify relevant ones (e.g., TDD, code review, debugging, execution workflows)
   - Include discovered skills/plugins as recommendations in the plan
9. **Generate implementation plan** saved to `.claude/plans/<ticket-id>/<ticket-id>-plan.md`:
   - **Header**: Goal, architecture approach, tech stack
   - **Phases** with human-in-the-loop checkpoints between major phases
   - **Tasks**: Bite-sized steps with exact file paths and test commands
   - **Acceptance criteria checklist** from the ticket
   - **Recommended skills/plugins**: List discovered skills and plugins relevant to execution
   - **Execution handoff**: Present available execution-related commands and skills the user has installed

## Anti-Patterns
- Do NOT read large source files inline — delegate to a codebase-analyzer sub-agent (if available)
- Do NOT run multiple sequential explorations in the main context — parallelize via sub-agents
- Do NOT deep-dive into tangential areas — stay scoped to what the ticket requires
- If research exceeds budget without convergence, pause and ask the developer
