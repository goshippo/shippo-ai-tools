---
name: evaluate-alert
description: Evaluate noisy New Relic alerts using decision tree from Alert Noise Remediation
---

# Alert Evaluation Command

## Purpose
Pair with a developer to evaluate a New Relic alert and determine remediation steps using the decision tree from [Alert Noise Remediation - Technical Details](https://shippo.atlassian.net/wiki/spaces/CET/pages/4449140783).

## Input Required

> **Note:** The New Relic MCP cannot access alert URLs directly. Always ask for the details below.

- **Alert condition name** (required)
- **NRQL query** (required)
- **Threshold and evaluation window** (required)
- **incident.io data** (required) - Use `ee-scripts/` helpers or incident.io API with user's env var token; fall back to CSV export

If the user only provides a New Relic link, ask them to open it and provide:
1. The alert condition name
2. The NRQL query from the condition
3. The threshold (e.g., "> 1.0 for 2 minutes")
4. Is there a Runbook URL attached to the condition?
5. Do you have alert data from incident.io, or should we query it programmatically?

## Workflow

### Step 1: Gather Alert Details

Ask the developer for:
- Alert condition name
- NRQL query
- Threshold and evaluation window
- **Is there a Runbook URL attached to the alert condition?** (Yes/No)

Once provided, use `mcp__new-relic__run_nrql_query` to execute the query (adjust time range to recent data) and understand current behavior.

### Step 2: Gather and Validate Incident History

**Preferred:** Use scripts in `ee-scripts/` to query incident.io programmatically (the user's `INCIDENT_IO_API_KEY` env var provides authentication). If scripts don't cover the need, use the incident.io REST API directly.

**Fallback:** Ask the developer to provide incident history from incident.io as CSV export.

**Important:** Before analyzing, check the date range of the data:
- Look at earliest and latest "Last occurrence" dates
- Note the actual timespan covered (e.g., "28 days" vs "12 months")
- This affects how we answer Step 0c
- **If data covers less than 12 months, ask the user to provide additional historical data or confirm they don't have more**

**Important:** Separate alerts by severity/condition before computing stats. If multiple conditions exist for the same service (e.g., Warning vs Critical, or duplicate conditions in different policies), analyze each independently. Conflating them produces misleading decline rates.

Key data needed:
- How many times has this alert fired?
- How many were declined vs accepted?
- What's the typical resolution pattern (auto-resolved vs manual)?
- **What time period does this data cover?**

**CSV Parsing (required):** When CSV data is provided, always use `grep` to extract the exact row by alert condition name. Do not eyeball CSV data from file contents—this causes parsing errors. Make sure you double check your calculations to ensure correctness

### Step 3: Search for Context

**Run multiple parallel searches to find related tickets and documentation:**

#### 3a: JQL Exact Match (most reliable)
Search with the exact condition name in quotes:
```
summary ~ "\"Exact Alert Condition Name\""
```

#### 3b: JQL Full-Text Search (broader)
Search across all text fields:
```
text ~ "key phrase from condition name"
```

#### 3c: Rovo Unified Search (fuzzy/semantic)
Use `mcp__atlassian__search` with natural language query combining:
- Alert condition name
- Key terms like "alert", "error rate", endpoint names

> **Why multiple searches?** JQL `summary ~` only matches word stems, not exact phrases. A ticket titled "Alert: Legacy Rating API" won't match `summary ~ "POST Shipments Error Rate"`. Rovo search uses semantic matching which catches related content.

#### 3d: Confluence Search
Search for runbooks and postmortems:
```cql
text ~ "Alert Condition Name" OR text ~ "key endpoint"
```

**What to look for:**
- Existing tuning tickets (check status - may already be in progress!)
- Runbooks for this alert (note: if found but not attached to alert condition, recommend attaching as follow-up)
- Previous incidents or postmortems
- Historical tuning attempts
- Parent epics for alert noise reduction work

### Step 4: Walk Through Decision Tree

> **Approach:** Show your reasoning transparently in chat. Make decisions when evidence is conclusive. Stop and ask for clarification only when information is unknown or ambiguous. After reaching a classification, ask the developer for feedback on any inflection points they might disagree with.

---

#### Step 0: Fundamental Value Assessment

**0a: Is this alert tied to customer-facing impact or SLOs?**

**0b: Is the alert actionable (requires human intelligence to respond)?**
- Check: Does the runbook say "just monitor" or does it have specific actions?

**0c: Has this alert ever led to a customer-impacting incident in the past 12 months?**
- ⚠️ If incident.io data covers less than 12 months, note this and ask the developer for historical context

> If ALL THREE are "No" → **🔴 REMOVE** without further tuning.
> *"If it has never caused a customer outage in 12 months, it is not an alert—it is a metric."*

---

#### Q1: Does this alert indicate a USER-FACING SYMPTOM (not just an internal cause)?
- **No (cause-based)** → **🔴 REMOVE:** Change to symptom-based or make informational
- **Yes** → Continue to Q2

#### Q2: Can you take IMMEDIATE ACTION to resolve it?
- **No** → **🔴 REMOVE:** Convert to dashboard metric
- **Yes** → Continue to Q3

#### Q3: Does it fire during NORMAL OPERATIONS?
- **Yes (fires during normal ops)** → Continue to Q4
- **No (only fires during real issues)** → Skip to Q5

**Q3b: Are alerts auto-resolving before human review?**
- Check resolution times — consistent short durations (e.g., all ~5 min) suggest auto-resolution via NR condition clearing
- If yes, note this: auto-resolved alerts may mask whether the alert is actually actionable (nobody gets a chance to act)
- This is evidence toward **🟡 TUNE** (extend evaluation window) or **🔴 REMOVE** (convert to dashboard metric)

#### Q4: Does it CORRELATE with actual incidents when it fires?
- **Rarely/Never correlates** → **🔴 REMOVE:** Invalid metric or incorrect threshold
- **Yes, usually correlates** → Continue to Q5

#### Q5: Is the PRIORITY LEVEL appropriate for the business impact?
- **No** → **🟡 TUNE:** Correct priority level
- **Yes** → Continue to Q6

#### Q6: Is the EVALUATION WINDOW appropriate for the metric?
- **No** → **🟡 TUNE:** Adjust evaluation window
- **Yes** → Continue to Q7

#### Q7: Does it have clear OWNERSHIP and RUNBOOK?
- **No** → **🟡 IMPROVE:** Add ownership and response procedures
- **Yes** → **✅ KEEP:** Effective alert

---

### Step 5: Present Findings and Get Feedback

After walking through the decision tree, present:
1. Your reasoning at each step
2. The classification reached
3. Key inflection points where you made judgment calls

Ask: *"Do you disagree with any of these assessments?"*

### Step 6: Discuss Remediation Options

Based on classification, discuss specific remediation options:

**🔴 REMOVE options:**
- Delete the alert condition
- Convert to dashboard-only metric (no alerting)
- Rewrite as symptom-based alert

**🟡 TUNE options:**
- Adjust threshold (raise if too sensitive)
- Extend evaluation window (filter transient spikes)
- Add environment filter (`WHERE environment = 'production'`)
- Add attribute filter to exclude known benign patterns
- Change aggregation (percentile instead of average)
- Adjust priority/severity level

**🟡 IMPROVE options:**
- Assign team ownership
- Create or update runbook
- **Attach runbook URL to alert condition** (if runbook exists but isn't linked)
- Add response procedures

**✅ KEEP:**
- Document current state
- Continue monitoring decline rate

## Output Artifacts

After the decision tree evaluation and remediation discussion, offer to create:
1. **Jira comment** — Succinct summary of findings and recommendation for the alert's ticket (use jira-comments skill)
2. **Slack post** — Team outreach message for the alert's owning channel, formatted as initial post + thread with stats
3. **Tuning ticket** — If classification is TUNE or IMPROVE, offer to create a follow-up Jira ticket

## Output Format

After walking through the decision tree, summarize your analysis and ask for feedback:

| Question | Answer | Reasoning |
|----------|--------|-----------|
| Data timespan | | (e.g., "28 days: Nov 17 - Dec 14") |
| Runbook attached? | | (Yes / No - exists in Confluence / No - doesn't exist) |
| 0a: Customer impact/SLOs? | | |
| 0b: Actionable? | | |
| 0c: Customer incident in 12mo? | | |
| Q1-Q7 as needed... | | |

**Classification:** [REMOVE / TUNE / IMPROVE / KEEP]

**Recommended Action:** [specific next step]

## References
- [Alert Noise Remediation - Technical Details](https://shippo.atlassian.net/wiki/spaces/CET/pages/4449140783)
