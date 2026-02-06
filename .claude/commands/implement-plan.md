# Implement Plan Command

## Purpose
Execute an approved technical plan from a `CET-{id}-plan.md` file with structured, incremental progress.

## Usage
Provide the plan file path (e.g., `CET-1643-plan.md`)

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

### 3. Create Todo List
Break down the plan into small, achievable tasks:
- Each task should be completable in one focused effort
- Group tasks into phases matching the plan's structure
- Include verification steps
- Note dependencies between tasks
- Flag any ambiguities or unclear requirements

**STOP**: Present the todo list and ask for approval before proceeding

### 4. Clarify Ambiguities
Before implementation, raise questions about:
- Unclear requirements or acceptance criteria
- Missing technical details
- Potential conflicts with existing code
- Scope boundaries

**STOP**: Wait for clarification responses

### 5. Execute Phases
For each phase in the plan:
- Implement the changes
- Run relevant tests/checks
- Update the plan file: mark completed items `- [x]`, note deviations
- Provide brief phase summary (under 200 words)

**STOP**: Present phase results and get approval before proceeding to next phase

### 6. Verification
After each phase:
- Run `make check test` or project-equivalent
- Confirm success criteria met
- Document any deviations from plan
- Update the plan file with current status so context can be resumed if needed

### 7. Draft PR
After all phases are complete and tests pass:
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
