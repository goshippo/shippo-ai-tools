# Shippo AI Tools

Centralized AI skills, commands, and rules for sharing across engineering teams. Works with Claude Code, Cursor, and Warp.

## Quick Start

### Claude Code (Plugin Marketplace)

**Recommended:** Install via Claude Code's plugin marketplace:

```bash
# Option 1: Install directly from GitHub (recommended)
/plugin marketplace add goshippo/shippo-ai-tools
/plugin install shippo-skills@shippo-tools

# Option 2: Install from local clone
git clone git@github.com:goshippo/shippo-ai-tools.git ~/Projects/shippo-ai-tools
/plugin marketplace add ~/Projects/shippo-ai-tools
/plugin install shippo-skills@shippo-tools
```

Then invoke skills by name: `/review-pr`, `/plan-ticket`, etc.

> **Note:** This is a private repo. For auto-updates when installing from GitHub, set `GITHUB_TOKEN` in your environment with repo read access.

### Cursor / Warp

For non-Claude Code tools, clone the repository and reference files directly:

```bash
cd ~/Projects
git clone git@github.com:goshippo/shippo-ai-tools.git
```

- **Cursor**: Copy desired rules to your project's `.cursorrules` or reference via `@`
- **Warp**: Reference files in your Warp rules configuration

## Directory Structure

```
shippo-ai-tools/
├── .claude-plugin/
│   └── marketplace.json     # Marketplace metadata (skill bundles defined here)
├── plugins/
│   └── shippo-skills/
│       ├── skills/          # Skills for specific workflows and domain knowledge
│       │   ├── review-pr/
│       │   │   └── SKILL.md
│       │   ├── plan-ticket/
│       │   │   └── SKILL.md
│       │   └── ...
│       └── rules/           # Behavioral rules applied across all interactions
│           ├── communication_style.md
│           ├── writing_standards.md
│           └── engineering_work_taxonomy.md
└── README.md
```

Each skill is a directory with `SKILL.md` as the entrypoint. Optional supporting files include `template.md`, `examples/`, and `scripts/`.

### Skills vs Rules

| Type | Trigger | Purpose | Example |
|------|---------|---------|---------|
| **Skill** | User invokes (`/skill-name`) or AI detects relevant context | Specific workflows and domain knowledge | `/review-pr`, `jira` |
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
| `jira` | Jira ticket operations: create tickets, write comments, validate completion. Consolidated skill with templates and taxonomy reference. | Auto-invoked for Jira work |
| `creating-pull-requests` | PR creation with repo convention discovery and CI body-parsing safety | Auto-invoked when creating PRs |
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
2. Add your skill directory to `plugins/shippo-skills/skills/`:
   ```bash
   mkdir -p plugins/shippo-skills/skills/my-skill
   touch plugins/shippo-skills/skills/my-skill/SKILL.md
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

**Claude Code plugin users:** Updates are automatic when installed from GitHub (requires `GITHUB_TOKEN`).

**Manual clone users:** Pull latest changes periodically:

```bash
cd ~/Projects/shippo-ai-tools  # or wherever you cloned
git pull origin main
```

## Resources

- [ADR: Sharing AI Tools](https://shippo.atlassian.net/wiki/spaces/CET/pages/4554063873)
- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Cursor Rules](https://cursor.com/docs/context/rules)

## Ownership

Maintained by the Engineering Enablement team. Questions? Reach out in [#help-engineering-enablement](https://shippo.slack.com/archives/C0730C6KH7A).
