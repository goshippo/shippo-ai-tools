# Engineering Work Taxonomy Reference

## Sources of Truth

**Always fetch current options dynamically.** Values may change.

```
mcp__atlassian__getJiraIssueTypeMetaWithFields
  cloudId: "bcd1ff35-af4b-43de-88a4-a2bc11d9f807"
  projectIdOrKey: "<project key>"
  issueTypeId: "<issue type ID>"
```

Parse `allowedValues` for `customfield_11173`. Skip values containing "(deprecated)".

Values may differ by project and issue type. Always query for the specific project/issue type.

## Fallback Values (last verified: 2026-02-06)

Use only if dynamic lookup fails.

| Value | Jira Option ID |
|-------|---------------|
| Technical Investment (Tech Inv) | 11396 |
| Feature - Product Enhancement | 11480 |
| Carrier Compliance (Cc) | 11477 |
| Level of Effort (LoE) | 11338 |
| Partner Commitment (Pc) | 11478 |
| Team Management (TM) | 10775 |
| Inquiry - #help- channel | 12010 |
| Incident | 12011 |
| Incident Follow-up | 12012 |
| Security Patch | 12013 |
| Telemetry Investigation | 12014 |
| Toil | 12015 |
| System Patch | 12016 |
| Other | 12017 |

## Deprecated (Do Not Use)

- Unplanned (Bug, Incident, Inquiry)
- Run the Business (RIB)
- Change the Business (CtB)
- Developer Productivity

## Defaults by Issue Type

| Issue Type | Default Taxonomy | Rationale |
|------------|-----------------|-----------|
| Initiative | Feature - Product Enhancement | Strategic objectives are usually product features |
| Epic | Inherits from Initiative context | Match the parent Initiative's intent |
| Story | Feature - Product Enhancement | User-facing work |
| Task | Technical Investment (Tech Inv) | Most internal/technical tasks |
| Sub-task | Inherit from parent | Match the parent Task/Story taxonomy |
| Bug | System Patch | One-off fix for system health |

## Task Override Scenarios

Tasks default to Technical Investment. Override when context clearly indicates:

| Context | Override Taxonomy | Label |
|---------|------------------|-------|
| Post-incident remediation | Incident Follow-up | `incident-follow-up` |
| Active incident response | Incident | `incident` |
| On-call requests / help channel | Inquiry - #help- channel | — |
| Alert triage from NR/Databricks | Telemetry Investigation | — |
| Manual ops (scaling, config) | Toil | — |
| Carrier-mandated changes | Carrier Compliance (Cc) | — |
| Partner-specific request | Partner Commitment (Pc) | — |
| Security vulnerability fix | Security Patch | — |
| Scoping/estimation | Level of Effort (LoE) | — |

## Sub-task Inheritance Rule

Sub-tasks always inherit taxonomy from their parent Task or Story. Do not override unless explicitly instructed.

## Confluence Reference

[Engineering Work Taxonomy](https://shippo.atlassian.net/wiki/spaces/CRND/pages/3668508685/Engineering+Work+Taxonomy) — page ID `3671326832` in ENGG space.
