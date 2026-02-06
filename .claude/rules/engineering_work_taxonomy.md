# Engineering Work Taxonomy

## Sources of Truth

**Always fetch current taxonomy options dynamically.** Do not rely solely on this file — options may change.

1. **Jira field metadata** (primary): Query `customfield_11173` allowed values via:
   ```
   mcp__atlassian__getJiraIssueTypeMetaWithFields
     cloudId: bcd1ff35-af4b-43de-88a4-a2bc11d9f807
     projectIdOrKey: <project key, e.g. "CET">
     issueTypeId: <issue type ID>
   ```
   Parse the `allowedValues` array for `customfield_11173` — each entry has `value` (display name) and `id` (for programmatic use). Skip any value containing "(deprecated)".

2. **Confluence documentation** (context): [Engineering Work Taxonomy](https://shippo.atlassian.net/wiki/spaces/CRND/pages/3668508685/Engineering+Work+Taxonomy) — page ID `3671326832` in ENGG space. Contains admin links for field configuration.

**Note:** Allowed values may differ by project and issue type. Always query for the specific project/issue type you're creating.

## Fallback Reference (last verified: 2026-02-06)

Use this only if dynamic lookup fails. Verify against Jira before relying on these.

| Value | Jira Option ID |
|-------|---------------|
| Technical Investment (Tech Inv) | 11396 |
| Carrier Compliance (Cc) | 11477 |
| Feature - Product Enhancement | 11480 |
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

## Selection Guide

- Internal tooling, refactors, test coverage, dependency updates → **Technical Investment**
- New carrier integration or product feature → **Feature - Product Enhancement**
- Carrier-mandated changes (API updates, compliance) → **Carrier Compliance**
- Scoping/estimating future work → **Level of Effort**
- Partner-specific request → **Partner Commitment**
- Alert triage or investigation → **Telemetry Investigation**
- Incident response or follow-up → **Incident** or **Incident Follow-up**
- Team ops (hiring, reviews, on-call) → **Team Management**

## Deprecated (Do Not Use)

- Unplanned (Bug, Incident, Inquiry)
- Run the Business (RIB)
- Change the Business (CtB)
- Developer Productivity
