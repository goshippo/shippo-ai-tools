---
name: address-pr-comments
description: Analyze and address all review comments on a pull request
allowed-tools: Bash(gh *, git *), Read, Grep, Task
---

# Analyze and Address PR Comments

You are tasked with analyzing and addressing all review comments on a pull request.

## Step 1: Fetch PR Comments

### Option A: Fetch Only Unresolved Comments (Recommended)

Use GitHub's GraphQL API to fetch only unresolved, non-outdated review comments:

```bash
PR_NUM=$(gh pr view --json number -q .number) && \
OWNER=$(gh pr view --json headRepositoryOwner -q .headRepositoryOwner.login) && \
REPO=$(gh pr view --json headRepository -q .headRepository.name) && \
gh api graphql -f query="{ repository(owner: \"$OWNER\", name: \"$REPO\") { pullRequest(number: $PR_NUM) { reviewThreads(first: 100) { edges { node { isResolved isOutdated comments(first: 10) { nodes { author { login } body path line createdAt } } } } } } } }" --jq '.data.repository.pullRequest.reviewThreads.edges[] | select(.node.isResolved == false and .node.isOutdated == false) | .node.comments.nodes[] | {author: .author.login, body: .body, path: .path, line: .line, created: .createdAt}'
```

### Option B: Fetch All Review Comments

Use the REST API to fetch all review comments (includes resolved and unresolved):

```bash
gh api repos/:owner/:repo/pulls/$(gh pr view --json number -q .number)/comments --jq '.[] | select(.side) | {author: .user.login, body: .body, path: .path, line: .line, created: .created_at}'
```

## Step 2: For Each Comment, Perform Analysis

For every comment retrieved, perform the following analysis:

### 2.1 Evaluate the Comment
- **Read the context**: Read the file at the specified path and examine the code around the mentioned line
- **Understand the concern**: Determine what the reviewer is pointing out (bug, style issue, performance concern, security issue, etc.)
- **Assess validity**: Evaluate whether the comment makes sense given the code context

### 2.2 Determine if Change is Required
- **Does it require a code change?** (vs. just informational or a question)
- **Is it valid?** (vs. based on misunderstanding)
- **Is it in scope?** (vs. out of scope for this PR)
- **What's the priority?** (critical, important, nice-to-have)

### 2.3 Present Numbered Analysis

Present ALL comments in a numbered list with this format per comment:

```
### Comment N: [short description]
**Reviewer**: @username | **File**: path/to/file:line
**Assessment**: [change required / no change needed / needs clarification / needs investigation]
**Reasoning**: [1-2 sentences on why]
**Suggested action**: [what you'd do if approved]
```

After presenting ALL comments, ask the user to respond with numbered decisions. Example prompt:
> "Respond with your decision per comment number (e.g., '1. agree, make the change  2. skip  3. dig deeper on this')"

**Do NOT make code changes or post comments until the user gives direction.**

### 2.4 Execute User Decisions

After receiving numbered decisions, process each:

- **"agree" / "make the change"**: Implement the code change
- **"skip" / "not necessary"**: Move on, no action
- **"dig deeper" / "investigate"**: Spawn a sub-agent to research (see Step 2.5)
- **"draft a reply"**: Write a concise, postable comment (2-3 sentences max) — present in chat, do NOT post unless explicitly asked

Commit and push only when the user explicitly requests it.

### 2.5 Delegated Research (when needed)

When a comment requires deeper investigation (cross-repo references, proof/evidence, unclear reviewer intent), use the Task tool to spawn sub-agents instead of researching inline:

- **codebase-analyzer**: Understand how specific code works with file:line references
- **codebase-locator**: Find relevant files across repos (e.g., config in shippo-k8s-deploy, dashboards in shippo-tf-services)
- **web-search-researcher**: External docs, library behavior, best practices

Guidelines:
- Limit to **3-5 sub-agent dispatches** per session. If more needed, ask the user.
- Each sub-agent should return a **focused summary under 200 words**
- Spawn sub-agents in parallel when researching independent comments
- Do NOT read large source files inline — delegate to a sub-agent

## Step 3: Summary

After processing all comments, provide:
- Total comments analyzed
- Number of changes made
- Number of comments that need clarification
- Number of comments deemed not requiring action
- List of files modified

## Example Workflow

```
Comment from @reviewer:
File: src/utils/validator.py, Line: 42
"This validation could fail for edge cases with unicode characters"

Analysis:
✓ Makes sense - the current regex doesn't handle unicode properly
✓ Change required - this is a valid bug that should be fixed
✓ Priority: Important (potential bug)

Action taken:
- Updated regex pattern to use unicode-aware character classes
- Added test case for unicode input
- Files changed: src/utils/validator.py, tests/test_validator.py
```

## Notes
- Be thorough but efficient - don't over-engineer solutions
- Respect the reviewer's expertise, but use your judgment
- When in doubt, ask for clarification rather than making assumptions
- Keep changes focused on addressing the specific feedback
- Ensure all changes are tested and don't break existing functionality
- **Comment drafts** should be concise (2-3 sentences). Present them in chat for the user to copy/post — do NOT post comments to GitHub unless the user explicitly asks
