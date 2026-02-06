---
name: create-jira-ticket
description: Use when creating Jira tickets (Initiatives, Epics, Tasks, Stories, Sub-tasks). Applies standardized templates with type-specific taxonomy and label guidance. Includes duplication checks via Atlassian MCP tools.
---

# Creating Jira Tickets

## Core Principle

**Template adherence is mandatory.** Every ticket follows the exact structure from jira_templates.md. No exceptions.

> **Future**: This skill is a candidate for splitting into type-specific and/or team-specific templates (e.g., `templates/incident-follow-up.md`, `templates/cet-task.md`). For now, all conventions live here. When the skill exceeds ~300 lines or team-specific patterns diverge, extract into a template directory.

## Before Creating Any Ticket

```
1. Search existing tickets → Avoid duplication
2. Identify issue type → Initiative, Epic, Task, or Story
3. Determine parent → Link to Epic/Initiative if applicable
4. Gather context → Use Atlassian MCP tools for project data
```

### Duplication Check (Required)

```
mcp__atlassian__searchJiraIssuesUsingJql with:
- Similar title keywords
- Same project
- Recent timeframe (last 90 days)
```

If duplicates exist: reference them, don't create new tickets.

## Issue Type Selection

| Type | When to Use | Parent |
|------|-------------|--------|
| Initiative | Quarterly strategic objective | None |
| Epic | Key Result under Initiative | Initiative |
| Story | User-facing value, feature delivery | Epic |
| Task | Internal/technical/operational work | Epic (optional) |
| Sub-task | Breakdown of parent Task/Story | Task or Story |

**Story vs Task Decision:**
- Story: "As a [user], I want [feature] so that [value]" - has user-facing outcome
- Task: Backend ops, infrastructure, refactors, setup - no user-facing value

**Sub-task Rule:** Sub-tasks inherit taxonomy and labels from their parent issue. Do not override unless explicitly instructed.

## Title Format (Strict)

```
[Quarter/Year][Category][KR#] [Name]
```

| Component | Format | Examples |
|-----------|--------|----------|
| Quarter/Year | `[Q1/2025]` | `[Q3/2025]`, `[Q4/2026]` |
| Category | `[AI]`, `[INFRA]`, `[DATA]` | Project-specific tags |
| KR# | `[KR1]`, `[KR2]` | Key Result reference (if applicable) |
| Name | Concise description | 5-10 words max |

**Examples:**
- `[Q3/2025][AI][KR4] Promote AI Adoption`
- `[Q3/2025][INFRA][KR2] Setup Artifactory publishing for MCP servers`
- `[Q3/2025][AI][KR1] Prepare Claude PM demo for AI showcase`

## Description Standards

**2-3 sentences maximum.** No verbosity.

| Section | When to Include | Content |
|---------|-----------------|---------|
| Description | Always | Brief what/why in 2-3 sentences |
| Scope | When boundaries matter | What's included/excluded |
| Technical Details | Tasks only | Implementation specifics |
| Background | Only if context needed | Why this matters |
| Story Content | Stories only | Key elements/deliverables |

## Acceptance Criteria Rules

**1-3 criteria maximum.** If more needed, split the ticket.

| Rule | Rationale |
|------|-----------|
| Measurable | Can verify pass/fail |
| Outcome-focused | What, not how |
| Bullet points | Not numbered lists |
| No process steps | Results only |

**Good AC:**
- Dashboard displays 95th percentile latency under 200ms
- API returns valid response within 2 seconds
- Documentation published to Confluence

**Bad AC:**
- Run the tests
- Deploy to staging
- Review with team

## Templates

### Initiative/Epic

```markdown
**Description:**
[Initiative name] - [Team] team focus on:

* [Key objective 1]
* [Key objective 2]
* [Key objective 3]

**Key Results:**
* KR 1: [Ticket-###] - [KR Name]
* KR 2: [Ticket-###] - [KR Name]

**Resources:**
* Project Page: [Confluence Link]
```

### Epic (Key Result)

```markdown
**Description:**
[Brief objective description]. Focus on [primary impact area].

**Scope:** [Specific boundaries].

**AC:**
* [Measurable outcome 1]
* [Measurable outcome 2]
```

### Task

```markdown
**Description:**
[Brief task description]. [Context if needed].

**Scope:** [Included]. [Excluded].

**Technical Details:**
* [Requirement 1]
* [Requirement 2]

**AC:**
* [Deliverable 1]
* [Deliverable 2]
```

### Story

```markdown
**Description:**
[User story from user perspective]. This [action] will [value delivered].

**Background:** [Why this matters].

**Scope:** [What's included]. [Delivery format].

**Story Content:**
* [Element 1]
* [Element 2]

**AC:**
* [User-focused criteria 1]
* [User-focused criteria 2]
```

## Required Fields

| Field | Value |
|-------|-------|
| Issue Type | Initiative / Epic / Task / Story / Sub-task |
| Parent | Link to parent Epic/Initiative (Sub-tasks link to Task/Story) |
| Engineering Work Taxonomy | See type-specific guidance below |
| Labels | See type-specific guidance below |
| customfield_11173 | Taxonomy value from guidance below |

## Engineering Work Taxonomy by Issue Type

| Issue Type | Default Taxonomy | Notes |
|------------|-----------------|-------|
| Initiative | Feature - Product Enhancement | Strategic objectives are usually product features |
| Epic | Inherits from Initiative context | Match the parent Initiative's intent |
| Story | Feature - Product Enhancement | User-facing work is almost always a feature |
| Task | Technical Investment (Tech Inv) | Most internal/technical tasks. Override when context is clear (see below) |
| Sub-task | Inherit from parent | Match the parent Task/Story taxonomy |
| Bug | System Patch | One-off fix for system health |

### Task Taxonomy Overrides

When a Task clearly fits a non-default category, use the appropriate taxonomy:

| Context | Taxonomy | Label(s) |
|---------|----------|----------|
| Post-incident work | Incident Follow-up | `incident-follow-up` |
| Active incident response | Incident | `incident` |
| On-call requests / help channel | Inquiry - #help- channel | — |
| Alert triage from NR/Databricks | Telemetry Investigation | — |
| Manual ops (scaling, config) | Toil | — |
| Carrier-mandated changes | Carrier Compliance (Cc) | — |
| Partner-specific request | Partner Commitment (Pc) | — |
| Security fix | Security Patch | — |
| Scoping/estimation | Level of Effort (LoE) | — |

### Default Labels

| Context | Labels |
|---------|--------|
| Standard ticket | `ready-for-grooming` |
| Incident follow-up | `incident-follow-up` |
| Both apply | Include both labels |

## Incident Follow-up Tickets

When creating tickets from incident investigations:

1. **Taxonomy**: Set to "Incident Follow-up"
2. **Label**: Add `incident-follow-up`
3. **Reference incidents**: Include INC-xxxx IDs in description
4. **Link to parent**: If the follow-up belongs under an existing story/epic, link it
5. **Description pattern**:

```markdown
**Description:**
Follow-up from [INC-xxxx](link). [Brief description of the fix needed].

**Context:**
* Root cause: [1 sentence]
* Impact: [1 sentence]

**AC:**
* [Fix or mitigation deliverable]
```

## Writing Standards

### Avoid These Words
- leverage, utilize, enhance, streamline, optimize
- comprehensive, robust, pivotal, crucial
- delve, navigate, facilitate, unlock

### Use Instead
- use, improve, enable, help
- complete, strong, important, key
- explore, handle, make easier, access

### Anti-Patterns
- Verbose explanations
- Excessive context/background
- Process steps as AC
- Praise language ("great", "excellent")
- Hedging ("perhaps", "maybe consider")

## Pre-Creation Checklist

Before creating any ticket:

- [ ] Searched for duplicates using JQL
- [ ] Confirmed issue type (Story vs Task)
- [ ] Title follows `[Q#/YYYY][Category][KR#] [Name]` format
- [ ] Description is 2-3 sentences max
- [ ] AC is 1-3 measurable criteria max
- [ ] Parent epic/initiative linked (if applicable)
- [ ] Labels set per context (default: `ready-for-grooming`, incidents: `incident-follow-up`)
- [ ] Engineering Work Taxonomy set per issue type guidance above
- [ ] No blacklisted words in content
- [ ] No verbose explanations

## MCP Tools Reference

| Tool | Purpose |
|------|---------|
| `mcp__atlassian__searchJiraIssuesUsingJql` | Check for duplicates |
| `mcp__atlassian__getJiraIssue` | Get existing ticket details |
| `mcp__atlassian__createJiraIssue` | Create new ticket |
| `mcp__atlassian__getVisibleJiraProjects` | Find project keys |
| `mcp__atlassian__getJiraProjectIssueTypesMetadata` | Get valid issue types |
| `mcp__atlassian__lookupJiraAccountId` | Find assignee account IDs |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No duplication check | Search JQL first |
| 5+ acceptance criteria | Split into multiple tickets |
| Description paragraph | Cut to 2-3 sentences |
| Process steps as AC | Focus on outcomes |
| Missing KR reference | Add `[KR#]` to title |
| Task for user-facing work | Use Story instead |
| Story for internal work | Use Task instead |
| No parent link | Connect to Epic/Initiative |

## Workflow

```
1. User requests ticket creation
2. Search existing tickets (duplication check)
3. If duplicate exists → Report and stop
4. Determine issue type
5. Apply template structure
6. Write succinct content (2-3 sentences)
7. Define 1-3 measurable AC
8. Verify checklist items
9. Create via MCP tool
10. Return ticket key and link
```
