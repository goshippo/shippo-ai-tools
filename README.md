# Shippo AI Tools

Centralized AI skills, commands, and rules for sharing across engineering teams. Works with Claude Code, Cursor, and Warp.

## Quick Start

### Option 1: Clone alongside your projects

```bash
cd ~/Projects  # or your preferred location
git clone git@github.com:goshippo/shippo-ai-tools.git
```

Then reference configs in your AI tool:
- **Claude Code**: Use `@shippo-ai-tools/.claude/commands/review-pr.md` in prompts
- **Cursor**: Copy desired rules to your project's `.cursorrules` or reference via `@`
- **Warp**: Reference files in your Warp rules configuration

### Option 2: Symlink (optional)

```bash
git clone git@github.com:goshippo/shippo-ai-tools.git ~/shippo-ai-tools
ln -s ~/shippo-ai-tools/.claude ~/.claude
```

> **Note:** This replaces any existing `~/.claude` directory.

## Directory Structure

```
.claude/
├── commands/       # Slash commands (e.g., /review-pr, /plan-ticket)
├── skills/         # Contextual skills triggered by task type
└── rules/          # Behavioral rules applied across all interactions
```

### Commands vs Skills vs Rules

| Type | Trigger | Purpose | Example |
|------|---------|---------|---------|
| **Command** | User invokes explicitly (`/command`) | Specific workflow with defined steps | `/review-pr`, `/commit` |
| **Skill** | AI detects relevant context | Domain knowledge for specific tasks | `create-jira-ticket`, `querying-new-relic` |
| **Rule** | Always active | Behavioral constraints | `communication_style`, `writing_standards` |

## Available Configs

### Commands
| Command | Description |
|---------|-------------|
| `review-pr` | Thorough PR review with proactive deeper analysis, validation/observability checks, and existing comment assessment |
| `plan-ticket` | Deep-dive Jira ticket analysis with delegated sub-agent research, skill discovery, and phased implementation planning |
| `implement-plan` | Execute approved technical plans with phased implementation, human-in-the-loop gates, TDD, and draft PR creation |
| `improve-skill` | Analyze conversation history to find feedback patterns on a skill/command and suggest targeted updates |
| `address-pr-comments` | Analyze and address PR review comments with review-first workflow, sub-agent research, and concise comment drafting |
| `evaluate-alert` | Evaluate noisy New Relic alerts using decision tree |

### Skills
| Skill | Description |
|-------|-------------|
| `create-jira-ticket` | Standardized Jira ticket creation with templates |
| `creating-pull-requests` | PR creation with mandatory ticket references, repo convention discovery via sub-agent, and CI body-parsing safety |
| `jira-comments` | Succinct, direct Jira comments with draft-first workflow, brevity rules, and anti-pattern guards |
| `querying-new-relic` | NRQL query best practices and timeout prevention |

### Rules
| Rule | Description |
|------|-------------|
| `communication_style` | Substance over praise, default brevity, no conversation leak in external-facing writing, direct engagement |
| `writing_standards` | Blacklisted AI-isms with alternatives, filler phrase detection, top offender callouts |

## Contributing

### Adding a New Config

1. Create a branch: `git checkout -b add-my-config`
2. Add your file to the appropriate directory:
   - `commands/` for explicit workflows
   - `skills/` for contextual domain knowledge
   - `rules/` for behavioral constraints
3. Follow the format of existing files (frontmatter + content)
4. Open a PR with description of what the config does

### Config File Format

```markdown
---
name: config-name
description: Brief description of when this config applies
---

# Config Title

## Purpose
What this config does and when to use it.

## Content
The actual instructions, templates, or rules.
```

### Guidelines

- **Keep configs focused**: One responsibility per file
- **Be specific**: Vague instructions produce vague results
- **Include examples**: Show expected input/output when relevant
- **Test before contributing**: Verify the config works as intended
- **Update this README**: Add new configs to the tables above

## Updating

Pull latest changes periodically:

```bash
cd ~/shippo-ai-tools  # or wherever you cloned
git pull origin main
```

## Resources

- [ADR: Sharing AI Tools](https://shippo.atlassian.net/wiki/spaces/CET/pages/4554063873)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Cursor Rules](https://cursor.com/docs/context/rules)

## Ownership

Maintained by the Engineering Enablement team. Questions? Reach out in [#help-engineering-enablement](https://shippo.slack.com/archives/C0730C6KH7A).
