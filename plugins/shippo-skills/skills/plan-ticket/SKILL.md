---
name: plan-ticket
description: Deep-dive Jira ticket analysis and implementation planning
allowed-tools: ToolSearch(mcp__atlassian__*), Task, Read, Write, Grep, Glob
---

# Ticket Researcher Command

## Usage
When user provides a Jira ticket ID or URL, analyze it deeply and create an implementation plan.

## Steps
1. **Fetch ticket details** using `mcp__atlassian__getJiraIssue` with the ticket ID/key
2. **Get comments** using `mcp__atlassian__searchJiraIssuesUsingJql` to find related discussions
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
7. **Discover available skills** — Claude already has installed skill descriptions in context. Identify relevant ones for implementation (e.g., TDD, code review, debugging, execution workflows) and reference them by `/name` invocation.
8. **Generate implementation plan** saved to `.claude/plans/<ticket-id>/<ticket-id>-plan.md`:
   - **Header**: Goal, architecture approach, tech stack
   - **Phases** with human-in-the-loop checkpoints between major phases
   - **Tasks**: Bite-sized steps with exact file paths and test commands
   - **Acceptance criteria checklist** from the ticket
   - **Recommended skills**: List discovered skills relevant to execution
   - **Execution handoff**: Present available execution-related skills the user has installed
9. **Wait for developer agreement** on the plan

## Anti-Patterns
- Do NOT read large source files inline — delegate to a codebase-analyzer sub-agent (if available)
- Do NOT run multiple sequential explorations in the main context — parallelize via sub-agents
- Do NOT deep-dive into tangential areas — stay scoped to what the ticket requires
- If research exceeds budget without convergence, pause and ask the developer
