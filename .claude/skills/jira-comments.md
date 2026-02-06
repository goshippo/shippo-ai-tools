# Jira Comments Skill

## Name
jira-comments

## Description
Write succinct, direct Jira ticket comments aligned with ticket context.

## When to Use
- Adding status updates to Jira tickets
- Documenting decisions or blockers
- Requesting clarification on requirements
- Noting progress against acceptance criteria
- Communicating with stakeholders via ticket comments

---

## Requirements Checklist

Before posting a Jira comment, verify:

- [ ] **Draft first** - Always show the full comment text to the user for approval before posting via API. Never post directly.
- [ ] **Direct opener** - First sentence states the point (no preamble)
- [ ] **AC alignment** - Comment references or advances acceptance criteria
- [ ] **No filler** - Remove "just wanted to", "I think", "it's worth noting"
- [ ] **No praise** - Skip "great idea", "thanks for the update", "perfect"
- [ ] **No blacklisted words** - Avoid delve, leverage, comprehensive, robust, etc.
- [ ] **Concrete details** - Include specific files, PRs, or next steps
- [ ] **No local paths** - Never reference local filesystem paths (e.g., `/Users/.../`). Use repo-relative paths instead (e.g., `shippo_py3/app/consumers/...`)
- [ ] **Single purpose** - One topic per comment (split if needed)

---

## Comment Structure

1. **State the update or question** - Lead with what matters
2. **Provide context** - Only if necessary for understanding
3. **List specifics** - Files, PRs, blockers, or decisions
4. **Next action** - What happens next (if applicable)

**Brevity rule**: Default to the shortest comment that conveys the information. If a comment exceeds 6 lines, look for what can be cut. Bullet points over paragraphs. No redundant context.

---

## Examples

### Status Update

Bad:
> Just wanted to give a quick update on this ticket. I've been delving into the codebase and have made some comprehensive changes to enhance the functionality. Great progress so far! Will continue to leverage existing patterns to ensure we deliver a robust solution.

Good:
> Completed AC #1 and #2. PR open: #4521
>
> Remaining: AC #3 requires API changes from platform team. Blocked until PLAT-892 merges.

### Requesting Clarification

Bad:
> Thanks for the detailed requirements! I have a quick question - it would be great if you could clarify what exactly you mean by "improved performance". Additionally, I'm wondering about the scope here.

Good:
> AC #2 says "improved performance" - need specifics:
> - Target latency? (current p99 is 450ms)
> - Specific endpoints, or all?
> - Acceptable tradeoffs (memory vs speed)?

### Documenting a Decision

Bad:
> After extensively exploring multiple approaches and having robust discussions with the team, we've decided to leverage the existing service rather than building a new one. This will help us streamline our architecture and optimize delivery timelines.

Good:
> Decision: Use existing `order-service` instead of new microservice.
>
> Reasons:
> - Already handles 80% of required logic
> - Adding endpoints takes ~2 days vs ~2 weeks for new service
> - Reduces operational overhead
>
> Trade-off: Slightly larger service scope. Acceptable per arch review.

### Noting a Blocker

Bad:
> Unfortunately, I've hit a bit of a roadblock. It seems like there might be some issues with the downstream API that's preventing us from moving forward. Just wanted to flag this for visibility!

Good:
> Blocked: `shipping-rates-api` returns 500 for weight > 150lbs.
>
> Steps to reproduce in ticket INC-2341. Waiting on platform team fix.
>
> ETA unknown. Will update when unblocked.

### Progress Against AC

Bad:
> Making great progress on this comprehensive ticket! I've been working through the acceptance criteria and things are looking good.

Good:
> Progress:
> - [x] AC #1: Auth middleware added (commit abc123)
> - [x] AC #2: Unit tests passing
> - [ ] AC #3: Integration tests - in progress
> - [ ] AC #4: Not started (depends on AC #3)
>
> On track for Thursday completion.

---

## References

All Jira comments must also follow:
- `@.claude/rules/writing_standards.md` — word choice (no blacklisted AI-isms)
- `@.claude/rules/communication_style.md` — tone, directness, no conversation leak
