---
name: querying-new-relic
description: Use when querying New Relic via MCP tools, writing NRQL queries, or investigating metrics and logs. Applies to incident investigations, performance analysis, or observability tasks.
---

# Querying New Relic

## Core Principle

**Start narrow, expand later.** Begin with tight filters and short time ranges. Expand only after confirming data exists.

## Required Filters

**Every query MUST include:**

```nrql
-- For Logs
WHERE environment = 'prod' AND cluster_name = 'prod-ixion'

-- For Metrics
WHERE cluster_name = 'prod-ixion'
```

**Never query without environment/cluster filters.** This causes timeouts and bloated responses.

## Time Range Rules

**If the user provides a specific time range, use it.** Only default to narrow ranges when exploring without guidance.

| Starting Point | When to Use |
|----------------|-------------|
| User-specified range | Always honor if provided |
| `SINCE 1 hour ago` | Default for exploratory queries |
| `SINCE 4 hours ago` | If 1 hour shows nothing |
| `SINCE 1 day ago` | Only after confirming data exists |

**Never query more than 1 day at once.** For multi-day analysis, run separate 1-day queries:

```nrql
-- Day 1
SELECT count(*) FROM Log WHERE ... SINCE 1 day ago UNTIL 0 days ago
-- Day 2
SELECT count(*) FROM Log WHERE ... SINCE 2 days ago UNTIL 1 day ago
-- Day 3
SELECT count(*) FROM Log WHERE ... SINCE 3 days ago UNTIL 2 days ago
-- ... up to 7 days
```

This avoids timeouts and produces more reliable data than a single 7-day query.

**If user says "last week":** Run 7 individual 1-day queries. Do not attempt a single 7-day query.

## Query Workflow

```
1. discover_event_attributes → Verify attribute names exist
2. Query with: 1 hour + required filters + LIMIT 10
3. If data exists → Expand time range or remove LIMIT
4. If empty → Check filters, try discover_event_types
```

## Timeout and Async Parameters

The MCP tool `run_nrql_query` accepts `timeout_seconds` and `use_async` parameters. Use them correctly:

**Default behavior (no overrides):**
- Connection timeout: 10s
- Read timeout: 120s (this is usually sufficient)
- Async auto-selects for: TIMESERIES, FACET, day/week ranges, queries >100 chars

**DO NOT set `timeout_seconds` to increase timeout.** Setting `timeout_seconds=30` replaces ALL timeouts (connect, read, write, pool) with 30s — this REDUCES the read timeout from the default 120s. Only set this if you need a value HIGHER than 120s.

**`use_async` has a 1-attempt polling limit.** If a query doesn't complete after the initial request + 1 poll, it fails. Async is not a fix for heavy queries — narrowing the query is.

| Situation | What to do |
|-----------|------------|
| Query times out | Narrow filters or time range. Do NOT add `timeout_seconds`. |
| Still timing out | Split into multiple smaller queries (see Time Range Rules). |
| Need >120s read timeout | Set `timeout_seconds` to desired value (e.g., 180). |

## Context Protection

**Always delegate NR queries to sub-agents when possible.** NR responses are large and exhaust the main context window.

**When to use sub-agents:**
- Any investigative query series (more than 1-2 queries)
- Multi-day trend analysis (each day = separate query in sub-agent)
- Exploratory queries where you don't know what you'll find
- When the user asks for NR data as part of a larger task (ticket review, RCA, etc.)

**Sub-agent setup:**
- Pass this skill's content to the sub-agent prompt
- Include the specific question to answer (not just "run queries")
- Have the sub-agent return a summary of findings, not raw query results

**When direct queries are fine:**
- Single, targeted query with known small result set
- User provides an exact query to run and wants the raw result

## Preventing Timeouts

| Cause | Prevention |
|-------|------------|
| Large time range | Start with `SINCE 1 hour ago` |
| Missing filters | Always include `cluster_name` and `environment` |
| No LIMIT on SELECT * | Add `LIMIT 10` for exploratory queries |
| TIMESERIES on large range | Use `TIMESERIES AUTO` or larger buckets (5 min, not 1 min) |
| Complex string operations | Limit `substring()` on large datasets |

## Valid NRQL Syntax

### Aggregation Functions (USE THESE)
```
count(*), sum(attr), average(attr), max(attr), min(attr)
percentile(attr, 50, 95, 99), uniqueCount(attr), rate(sum(attr), 1 minute)
```

### String Functions (USE THESE)
```
substring(attr, start, length), position(attr, 'search')
```

### Clauses (USE THESE)
```
SELECT ... FROM ... WHERE ... FACET ... SINCE ... UNTIL ...
TIMESERIES ... LIMIT ... ORDER BY ... COMPARE WITH ...
```

### NOT Valid NRQL (DO NOT USE)
```
GROUP BY        -- Use FACET instead
HAVING          -- Not supported, filter in WHERE
JOIN            -- Not supported
DISTINCT        -- Use uniqueCount() instead
COALESCE        -- Not supported
IFNULL          -- Not supported
CONCAT          -- Not supported
```

## Event Types Reference

| Event Type | Purpose | Required Filters |
|------------|---------|------------------|
| `Log` | Application logs | `environment`, `cluster_name` |
| `Metric` | Prometheus/OTEL metrics | `cluster_name` |
| `Transaction` | APM data | `appName` |

## Attribute Naming

```nrql
-- Logs: underscore naming
FROM Log WHERE pod_name LIKE 'trackapi%'

-- Metrics: dot notation
FROM Metric WHERE service.name = 'trackapi-shippo-armor'
```

## Query Templates

### Safe Starting Query (Logs)
```nrql
SELECT count(*) FROM Log
WHERE environment = 'prod'
  AND cluster_name = 'prod-ixion'
  AND message LIKE '%keyword%'
SINCE 1 hour ago
LIMIT 10
```

### Safe Starting Query (Metrics)
```nrql
SELECT average(metric_name) FROM Metric
WHERE cluster_name = 'prod-ixion'
SINCE 1 hour ago
TIMESERIES AUTO
```

### Expanding After Success
```nrql
-- After confirming data exists, expand:
SINCE 4 hours ago    -- or
SINCE 1 day ago      -- or
LIMIT 100            -- remove LIMIT 10
```

## Common Mistakes

| Mistake | Result | Fix |
|---------|--------|-----|
| No cluster/environment filter | Timeout, bloated response | Add required filters |
| `SINCE 7 days ago` as first query | Timeout | Start with `SINCE 1 hour ago` |
| `GROUP BY` | Syntax error | Use `FACET` |
| `DISTINCT column` | Syntax error | Use `uniqueCount(column)` |
| `SELECT *` without LIMIT | Huge response | Add `LIMIT 10` |
| Made-up function names | Syntax error | Check valid functions above |

## MCP Tools

| Tool | Purpose |
|------|---------|
| `discover_event_types` | List available event types |
| `discover_event_attributes` | **Use first** - verify attribute names |
| `run_nrql_query` | Execute query |

## Empty Results Warning

**Empty results usually mean the query is wrong, not that data doesn't exist.**

Before concluding "no errors found", progressively validate each filter:

```nrql
-- Step 1: Verify base data exists (keep required filters)
SELECT count(*) FROM Log
WHERE cluster_name = 'prod-ixion' AND environment = 'prod'
SINCE 1 hour ago

-- Step 2: Add ONE filter at a time to find which breaks it
SELECT count(*) FROM Log
WHERE cluster_name = 'prod-ixion' AND environment = 'prod'
  AND pod_name LIKE 'trackapi%'    -- Does this filter work?
SINCE 1 hour ago

-- Step 3: Add next filter
SELECT count(*) FROM Log
WHERE cluster_name = 'prod-ixion' AND environment = 'prod'
  AND pod_name LIKE 'trackapi%'
  AND message LIKE '%error%'       -- Does this filter work?
SINCE 1 hour ago
```

**Why progressive validation matters:**
```nrql
-- This returns nothing:
SELECT * FROM Log WHERE non_existent_column = 'value'

-- This returns data:
SELECT count(*) FROM Log WHERE cluster_name = 'prod-ixion'
```

Without progressive validation, you might conclude "no errors exist" when actually `non_existent_column` is the problem. Always verify each filter individually.

## Debugging

1. **Timeout?** → Reduce time range, add filters, add LIMIT

   **DO NOT retry the same query with `timeout_seconds` or `use_async` overrides.**
   These are not fixes for heavy queries. Narrow the query instead.
   If a query times out, it means too much data — not too little time.

2. **Syntax error?** → Check function/clause is in valid list above
3. **Empty results?** → **Do not assume no data exists.** Verify query is correct first.
4. **Too much data?** → Add `cluster_name`, `environment`, reduce time range
