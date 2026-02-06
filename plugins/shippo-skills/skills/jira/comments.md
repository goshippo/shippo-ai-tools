# Writing Jira Comments

## Rules

- **Draft first** — always show the full comment text to the user for approval before posting via API. Never post directly.
- **Direct opener** — first sentence states the point, no preamble
- **AC alignment** — reference or advance acceptance criteria when relevant
- **Concrete details** — include specific files, PRs, or next steps
- **No local paths** — use repo-relative paths (`shippo_py3/app/consumers/...`), never absolute paths
- **Single purpose** — one topic per comment; split if needed
- **Brevity** — default to the shortest comment that conveys the information. If a comment exceeds 6 lines, look for what to cut. Bullet points over paragraphs.

Writing standards and communication style rules apply (loaded separately).

## Structure

1. **State the update or question** — lead with what matters
2. **Provide context** — only if necessary for understanding
3. **List specifics** — files, PRs, blockers, or decisions
4. **Next action** — what happens next (if applicable)

## Examples

### Status Update

> Completed AC #1 and #2. PR open: #4521
>
> Remaining: AC #3 requires API changes from platform team. Blocked until PLAT-892 merges.

### Requesting Clarification

> AC #2 says "improved performance" — need specifics:
> - Target latency? (current p99 is 450ms)
> - Specific endpoints, or all?
> - Acceptable tradeoffs (memory vs speed)?

### Documenting a Decision

> Decision: Use existing `order-service` instead of new microservice.
>
> Reasons:
> - Already handles 80% of required logic
> - Adding endpoints takes ~2 days vs ~2 weeks for new service
> - Reduces operational overhead
>
> Trade-off: Slightly larger service scope. Acceptable per arch review.

### Noting a Blocker

> Blocked: `shipping-rates-api` returns 500 for weight > 150lbs.
>
> Steps to reproduce in ticket INC-2341. Waiting on platform team fix.
> ETA unknown. Will update when unblocked.
