---
name: improve-skill
description: Analyze conversation history to find feedback patterns on a skill/command and suggest targeted updates
allowed-tools: Read, Bash(python3), Write
---

# Improve Skill from Conversation History

## Usage
Analyze conversation history to find patterns of user feedback when a specific skill or command is used, then suggest minimal targeted updates.

```
/improve_skill <skill-or-command-name> [time-period]
```

Examples:
- `/improve_skill review-pr 14d`
- `/improve_skill plan_ticket 30d`
- `/improve_skill commit 7d`

## Input
- `$ARGUMENTS` format: `<name> [period]`
  - `name`: The skill or command name (e.g., `review-pr`, `plan_ticket`)
  - `period` (optional): Time window to search. Format: `Nd` for N days. Default: `14d`

## Process

### Step 1: Parse Arguments

Extract the skill/command name and time period from `$ARGUMENTS`.

If arguments are missing or unclear, ask the user to provide: the skill/command name and optionally a time period.

### Step 2: Locate the Skill or Command File

Search for the target file in this order:
1. `.claude/commands/<name>.md`
2. `.claude/commands/<name_with_underscores>.md`
3. `.claude/skills/<name>.md`
4. Check if it's a plugin skill (e.g., `superpowers:<name>`)

If not found, report the issue and ask the user to clarify the name.

Read the full contents of the skill/command file. This is the **current version** you'll be suggesting changes to.

### Step 3: Elicit User Memories

Before searching history, ask the user for **2 specific memories** of things they'd like improved about this skill/command. Use the AskUserQuestion tool with a single open-ended question:

> "Before I search conversation history, tell me 2 specific things you remember wanting to improve about this skill/command. These can be behaviors that annoyed you, missing capabilities, incorrect assumptions it makes, or anything else."

These memories serve as **anchors** - they focus the history search on what matters most to the user.

### Step 4: Search Conversation History

Write and run a Python script to search `~/.claude/history.jsonl` for sessions that invoked the target command/skill within the time period.

**What to extract:**

1. **Find matching sessions** in `history.jsonl` — filter by command name in `display` field and timestamp within period. Deduplicate by `sessionId`.

2. **Read each session transcript** from `~/.claude/projects/{project_dir}/{sessionId}.jsonl`. Derive `project_dir` from the `project` field: replace `/` with `-` and drop the leading `/` (e.g., `/Users/jdoe/Projects` → `-Users-jdoe-Projects`).

3. **Extract user feedback** — keep only user messages that represent corrections, follow-ups, or clarifications. Filter out system tags, command invocations, and messages under 20 characters.

4. **Output** — total sessions, sessions with feedback vs without, all feedback grouped by session with date.

### Step 5: Analyze Patterns

Review all extracted feedback alongside the user's 2 memories. Identify:

1. **Repeated corrections** - things the user has to fix or redirect multiple times
2. **Missing capabilities** - things the user frequently asks for that the skill doesn't do
3. **Wrong assumptions** - places where the skill assumes something incorrect
4. **Workflow friction** - steps that consistently need manual intervention
5. **Scope issues** - the skill does too much or too little in certain areas

Prioritize patterns that appear in multiple sessions or that align with the user's stated memories.

### Step 6: Present Findings and Proposed Changes

Present the analysis in this format:

```
## History Analysis

- **Sessions analyzed**: X (Y with feedback, Z without)
- **Time period**: [start] to [end]

## Patterns Found

### Pattern 1: [description]
- Seen in N sessions
- Example feedback: "[quote from user]"
- Aligns with user memory: [yes/no + which one]

### Pattern 2: [description]
...

## Proposed Changes

For each change, show:
1. **What**: One-sentence description
2. **Why**: Which pattern(s) and/or user memory it addresses
3. **Diff**: The specific text to add, remove, or modify in the skill file

### Change 1: [title]
**Why**: Addresses Pattern X and user memory about Y

```diff
- [old text from skill]
+ [new text for skill]
```

### Change 2: [title]
...
```

## Constraints

- **Minimal changes only** - suggest the fewest edits that address the most patterns
- **Do not rewrite** the skill/command - preserve existing structure and voice
- **Do not auto-apply** - present proposals for user approval
- **Evidence-based** - every change must trace back to a specific pattern or user memory
- **No speculative improvements** - only address issues found in actual feedback
- **Cap at 5 changes** - if more patterns exist, prioritize the top 5 by frequency and user memory alignment
