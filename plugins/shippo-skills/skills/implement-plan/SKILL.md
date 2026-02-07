---
name: implement-plan
description: Execute approved technical plans with phased implementation and human-in-the-loop gates
allowed-tools: Read, Write, Glob, Grep, Bash(git *, make *, gh pr *), Task, ToolSearch(mcp__atlassian__*), mcp__github-prs__create_pull_request
---

# Implement Plan Command

## Purpose
Execute an approved technical plan with structured, incremental progress.

## Usage
Provide the plan file path (e.g., `.claude/plans/CET-1643/CET-1643-plan.md`)

## Process

### 1. Initial Analysis
- Read the plan file completely
- Read the original Jira ticket for full context
- Identify all files and systems mentioned

### 2. Branch Setup
- Identify the repository's default branch (e.g., `main` or `master`)
- Pull latest changes on the default branch
- Create a new branch from the updated default branch
- Branch name format: `<ticket-id>/<short-description>` (e.g., `CET-1643/add-retry-logic`)

**STOP**: Confirm branch name and starting point with user

### 3. Create Todo List and Clarify Ambiguities
Break down the plan into small, achievable tasks:
- Each task should be completable in one focused effort
- Group tasks into phases matching the plan's structure
- Include verification steps and note dependencies
- Flag ambiguities, unclear requirements, or scope questions

**STOP**: Present the todo list and any questions. Wait for approval and clarifications before proceeding.

### 4. Execute Phases
For each phase in the plan:
- Implement the changes
- Run `make check test` or project-equivalent
- Confirm success criteria met
- Update the plan file: mark completed items `- [x]`, note deviations
- Provide brief phase summary (under 200 words)

**STOP**: Present phase results and get approval before proceeding to next phase

### 5. Draft PR
After all phases are complete and tests pass:
- Follow the `creating-pull-requests` skill for PR conventions and template discovery
- Create a draft pull request against the default branch
- PR description should summarize the changes and reference the Jira ticket

**STOP**: Share PR link for user review

## Key Principles
- Keep responses under 200 words
- Get explicit user approval between each phase
- Clarify before implementing
- Focus on one task at a time
- Follow test-driven development: write/update tests before implementation code
- If the `superpowers:test-driven-development` skill is available, use it for each implementation step
