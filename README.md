# Shippo AI Tools

Centralized AI skills, commands, and rules for sharing across engineering teams. Works with Claude Code, Cursor, and Warp.

## Quick Start

### Option 1: Clone alongside your projects

```bash
cd ~/Projects  # or your preferred location
git clone git@github.com:goshippo/shippo-ai-tools.git
```

Then reference configs in your AI tool:
- **Claude Code**: Use `@shippo-ai-tools/.claude/skills/review-pr` in prompts or invoke with `/review-pr`
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
├── skills/         # Skills for specific workflows and domain knowledge
│   ├── review-pr/
│   │   └── SKILL.md       # Main instructions
│   ├── plan-ticket/
│   │   └── SKILL.md
│   └── ...
└── rules/          # Behavioral rules applied across all interactions
```

Each skill is a directory with `SKILL.md` as the entrypoint. Optional supporting files include `template.md`, `examples/`, and `scripts/`.

### Skills vs Rules

| Type | Trigger | Purpose | Example |
|------|---------|---------|---------|
| **Skill** | User invokes (`/skill-name`) or AI detects relevant context | Specific workflows and domain knowledge | `/review-pr`, `create-jira-ticket` |
| **Rule** | Always active | Behavioral constraints | `communication_style`, `writing_standards` |

## Available Configs

### Skills
| Skill | Description | Invocation |
|-------|-------------|------------|
| `review-pr` | Thorough PR review focused on software engineering best practices and accurate assessment | `/review-pr` |
| `plan-ticket` | Deep-dive Jira ticket analysis and implementation planning | `/plan-ticket` |
| `implement-plan` | Execute approved technical plans with phased implementation and human-in-the-loop gates | `/implement-plan` |
| `improve-skill` | Analyze conversation history to find feedback patterns on a skill/command and suggest targeted updates | `/improve-skill` |
| `address-pr-comments` | Analyze and address all review comments on a pull request | `/address-pr-comments` |
| `evaluate-alert` | Evaluate noisy New Relic alerts using decision tree from Alert Noise Remediation | `/evaluate-alert` |
| `validate-ticket` | Validate Jira ticket completion against acceptance criteria | `/validate-ticket` |
| `create-jira-ticket` | Standardized Jira ticket creation with type-specific taxonomy and templates | Auto-invoked when creating tickets |
| `creating-pull-requests` | PR creation with repo convention discovery and CI body-parsing safety | Auto-invoked when creating PRs |
| `jira-comments` | Succinct, direct Jira comments with draft-first workflow | Auto-invoked for Jira comments |
| `archive-plan-files` | Archive plan directories for closed Jira tickets | `/archive-plan-files` |
| `querying-new-relic` | NRQL query best practices with timeout and time range guidance | Auto-invoked for New Relic queries |

### Rules
| Rule | Description |
|------|-------------|
| `communication_style` | Substance over praise, default brevity, direct engagement |
| `writing_standards` | Blacklisted AI-isms with alternatives |
| `engineering_work_taxonomy` | Dynamic Jira taxonomy discovery with fallback reference table |

## Contributing

### Adding a New Config

1. Create a branch: `git checkout -b add-my-skill`
2. Add your skill directory to `.claude/skills/`:
   ```bash
   mkdir -p .claude/skills/my-skill
   touch .claude/skills/my-skill/SKILL.md
   ```
3. Follow the format of existing skills (YAML frontmatter + markdown content)
4. Optional: Add supporting files (`template.md`, `examples/`, `scripts/`)
5. Open a PR with description of what the skill does

### Skill File Format

```markdown
---
name: my-skill
description: Brief description of when this skill applies and what it does
disable-model-invocation: true  # Optional: prevent auto-invocation, require /my-skill
allowed-tools: Read, Grep, Bash(git *)  # Optional: restrict tool access
---

# My Skill

## Purpose
What this skill does and when to use it.

## Instructions
Step-by-step instructions or behavioral patterns.
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
