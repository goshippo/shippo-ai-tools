---
name: creating-pull-requests
description: Use when asked to create a PR, create a draft PR, or create a pull request in any repository. Discovers repo-specific PR templates, contributing guides, and CI constraints before writing the description.
allowed-tools:
  - Glob
  - Grep
  - Read
  - Bash(gh pr *)
  - Bash(git *)
  - Bash(echo)
  - Task
  - mcp__github-prs__create_pull_request
---

# Creating Pull Requests

## Core Principle

**Discover the repo's PR conventions before writing anything.** Every repository may have templates, CI checks, or contributing guides that dictate how PR descriptions must be formatted. Ignoring these causes CI failures and rework.

## Before Creating Any PR

```
1. Discover PR conventions → Search for templates, guides, CI checks
2. Analyze conventions → Extract required sections, format rules, CI constraints
3. Gather commit context → Read branch commits, diff, and ticket references
4. Write description → Follow discovered conventions
5. Validate description → Check for bash-unsafe characters and required fields
6. Create PR → Push, create, verify CI passes
```

### Convention Discovery (Required)

Search the repo for PR description guidance in this order:

| Priority | Location | What to Look For |
|----------|----------|-----------------|
| 1 | `.github/pull_request_template.md` | Required sections, placeholders |
| 2 | `.github/PULL_REQUEST_TEMPLATE/` | Multiple templates by type |
| 3 | `CONTRIBUTING.md` or `.github/CONTRIBUTING.md` | PR description rules |
| 4 | `.github/workflows/*.yaml` | CI steps that parse PR body |
| 5 | Recent merged PRs | De facto format patterns |

```
# Discovery commands
Glob: .github/pull_request_template.md
Glob: .github/PULL_REQUEST_TEMPLATE/**/*.md
Glob: CONTRIBUTING.md
Glob: .github/CONTRIBUTING.md
Grep: pattern "pull_request.body" in .github/workflows/
```

If a template exists, use its structure. If CI parses the body, respect its constraints.

### Ticket Reference Extraction (Required)

Before writing the description, identify the ticket reference:

1. **Branch name**: Extract from patterns like `CET-1234/description` or `feature/CET-1234`
2. **Conversation context**: Check if the user mentioned a Jira ticket ID
3. **Commit messages**: Scan commits on the branch for ticket references
4. **Plan files**: Check if an implementation plan references a ticket

The ticket reference MUST appear in:
- The PR title (e.g., `fix(CET-1234): description`)
- The PR body as a Jira link (e.g., `https://shippo.atlassian.net/browse/CET-1234`)

If no ticket can be identified from any source, ask the user before creating the PR.

### CI Body Parsing Check (Required)

Many CI workflows extract data from PR descriptions. Common patterns:

| CI Pattern | Risk | Example |
|------------|------|---------|
| `echo "${{ github.event.pull_request.body }}"` | Bash injection from special chars | Jira ticket extraction |
| `grep` on PR body | Requires specific URL/text format | Ticket validation |
| JSON parsing of body | Requires valid structure | Automated changelogs |

**Search for these patterns:**

```
Grep: "pull_request.body" in .github/workflows/*.yaml
Grep: "github.event.pull_request" in .github/workflows/*.yaml
```

If found, read the workflow to understand what format the CI expects and what characters would break it.

## Bash-Unsafe Characters

When CI interpolates the PR body into shell commands via `echo "${{ github.event.pull_request.body }}"`, these characters break parsing:

| Character | Problem | Alternative |
|-----------|---------|-------------|
| `"` (double quote) | Closes the outer `echo "..."` quote | Remove or use single quotes |
| `` ` `` (backtick) | Triggers command substitution | Remove, spell out instead |
| `$()` or `${}` | Command/variable substitution | Remove the `$` |
| `\` (backslash) | Escape sequences | Remove or double it |
| `!` (in some shells) | History expansion | Rephrase |

### Safe PR Body Rules

When a repo's CI interpolates the PR body into bash:

1. **No backticks** — write `close_old_connections` not `` `close_old_connections` ``
2. **No double quotes** — write `Worker Connection Investigation` not `"Worker Connection Investigation"`
3. **No `$()`** — write `count guard` not `$(count)`
4. **No parentheses in freeform text near quotes** — `(tf)` after a `"` break causes `syntax error near unexpected token '('`
5. **Markdown links are safe** — `[CET-1815](https://...)` works because `()` stays inside the outer echo quotes if no `"` precedes it

### Testing Locally

Before creating the PR, validate the body is bash-safe:

```bash
# Simulate the CI extraction
echo "<paste PR body here>" | grep -oE "expected_pattern"
```

## PR Description Structure

### When Template Exists

Follow the template exactly. Fill in every section. Remove placeholder text.

### When No Template Exists

**Dispatch a sub-agent** (via Task tool, subagent_type: Explore) to discover the repo's PR conventions. The sub-agent should check sources in this order:

1. **Local files first** (fast, no API calls):
   - `CONTRIBUTING.md` or `.github/CONTRIBUTING.md` — PR description rules
   - `.github/pull_request_template.md` — required sections, placeholders
   - `.github/PULL_REQUEST_TEMPLATE/` — multiple templates by type

2. **Recent merged PRs** (only if local files yield nothing):
   - `gh pr list --state merged --limit 5 --json number,title`
   - `gh pr view <number> --json body --jq '.body'` for the top 3

```
Sub-agent prompt:
"In this repo, check for CONTRIBUTING.md, .github/CONTRIBUTING.md,
.github/pull_request_template.md, and .github/PULL_REQUEST_TEMPLATE/.
If none exist or they don't specify PR format, run
`gh pr list --state merged --limit 5 --json number,title` then
`gh pr view <number> --json body --jq '.body'` for the top 3.
Report back: what sections they use, what format ticket references take,
whether they include test plans, and any other structural patterns."
```

Use the sub-agent's findings to build the description matching the repo's style.
If the sub-agent finds no conventions from any source, fall back to:

```
## Summary
<1-3 bullet points describing what changed and why>

## Test plan
<How to verify the changes>
```

## Workflow

```
1. User requests PR creation
2. Discover repo conventions (template, CI, contributing guide)
3. Check CI workflows for body parsing constraints
4. Extract ticket reference (branch name, conversation, commits, plan files)
5. Read branch diff and commit history
6. If no template found: dispatch sub-agent to analyze local convention files and recent merged PRs
7. Draft description following discovered conventions and matching repo style
8. Validate: no bash-unsafe characters (if CI parses body)
9. Validate: all required sections present (if template exists)
10. Validate: ticket reference in title and body
11. Push branch, create PR
12. Monitor CI checks — if body-parsing check fails, fix and update
```

## Pre-Creation Checklist

Before creating any PR:

- [ ] Searched for PR template in `.github/`
- [ ] Searched for `CONTRIBUTING.md`
- [ ] Checked CI workflows for PR body parsing
- [ ] Branch pushed to remote with `-u` flag
- [ ] Description follows repo template or merged PR patterns
- [ ] No bash-unsafe characters (if CI interpolates body)
- [ ] Required ticket links present in correct format
- [ ] Commit message follows repo conventions (check recent `git log`)

## Tools Reference

| Tool | Purpose |
|------|---------|
| `Glob` | Find PR templates and contributing guides |
| `Grep` | Search CI workflows for body parsing |
| `Read` | Read templates and workflow files |
| `Bash: gh pr create` | Create the PR |
| `Bash: gh pr view` | Check recent PR formats |
| `Bash: gh pr checks` | Verify CI passes |
| `mcp__github-prs__create_pull_request` | Create PR via MCP |

## Tool-Specific Formatting

Both the MCP tool and `gh` CLI work for creating PRs. Each has a newline pitfall.

### MCP Tool (mcp__github-prs__create_pull_request)

The `body` parameter accepts actual newlines. Do NOT use escaped `\n` sequences — they get double-escaped and render as visible `\n` on GitHub.

```
# BAD — escaped newlines become literal \n on GitHub
body: "## Summary\n\n- Change one\n- Change two"

# GOOD — actual newlines in the parameter value
body: "## Summary

- Change one
- Change two"
```

### gh pr create (Bash)

Use a HEREDOC to preserve newlines:

```bash
gh pr create --title "title" --body "$(cat <<'EOF'
## Summary
- Change one

## Test plan
- [x] Tests pass
EOF
)"
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping template discovery | Always search `.github/` first |
| Backticks in PR body | Remove when CI parses body via bash |
| Double quotes in PR body | Remove when CI uses `echo "${{ body }}"` |
| Missing ticket links | Check CI for required URL patterns |
| Ignoring template sections | Fill every section or remove with explanation |
| Not checking CI after creation | Always run `gh pr checks` and verify |
| Guessing PR format | Check 3-5 recent merged PRs for actual patterns |
| Using custom format when template exists | Template takes precedence |
| Escaped `\n` in MCP body parameter | Use actual newlines in the body value, not `\n` escape sequences |
