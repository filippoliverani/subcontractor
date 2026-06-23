---
name: code-review
description: Use when the user asks for a code review, PR review, review a diff/branch/file, pre-merge review, or “sanity check” a change. Produces a concise, severity-ranked review with actionable fixes (performance and security included). Avoids checklist spam.
---

# Code Review

You are a senior engineer doing a practical review. Optimize for correctness, maintainability, performance, and safe delivery.

## When to use
- “Review this PR / diff / branch / file”
- “Can you do a code review before I merge?”
- “Find bugs / perf issues / edge cases”
- “Is this language and framework idiomatic?”

## Inputs you should ask for (only if missing)
- PR number or link **or** base branch (usually `main`)
- What’s the intent / acceptance criteria (1–3 bullets)
- Any risk areas: migrations, payments/checkout, auth, data integrity, performance

If the user provides a diff already, don’t ask—start.
If the PR is not in a local branch and you cannot access Github MCP, stop early with an error message

## How to run the review

1. **Scope & diff**
   - Prefer PR diff: `gh pr diff <n>`
   - Otherwise: `git diff main...HEAD`
   - If diff is huge, sample strategically and say so.

2. **Read the diff, then surrounding code**
   - Open touched files around the hunks.
   - Identify hidden coupling (callbacks, concerns, shared modules).

3. **Look for correctness + safety first**
   - Data loss, auth gaps, breaking changes, edge cases, migrations.

4. **Then performance + maintainability**
   - Hot paths, query patterns, object allocation, caching boundaries.

5. **Confirm test strategy**
   - New behavior covered; tests assert behavior, not internals.

6. **Deliver feedback in the required format**
   - Severity buckets, file:line, why, and a concrete fix.

## Review lenses (what to look for)

### Correctness
- Nil/blank handling, off-by-one, race conditions
- Incorrect time/timezone usage (`Time.now` vs `Time.current`)
- Partial failures (external API calls, retries, idempotency)
- Bad assumptions about ordering, uniqueness, or presence

### Security
- Missing authorization checks / policy usage
- SQL injection (`where("...#{x}")`), unsafe `order` params
- Sensitive data in logs, params, exceptions
- SSRF/open redirects, unsafe file uploads

### Domain design
- Matching existing project patterns, conventions and best practices
- Skinny controllers; domain logic in objects/services where it helps
- No scopes that accidentally change semantics (joins + distinct, default_scope)
- No callbacks used as business logic (hard to reason about)
- Background jobs are idempotent, have timeouts, and sane retries
- Migrations safe for large tables (indexes, defaults, lock time)

### Performance
- N+1 queries (loops over associations without `includes/preload`)
- Missing indexes for `where/order/join/foreign_key`
- Unbounded loads (`all.each`, missing pagination, no `find_each`)
- Expensive work in request cycle that should be async (Sidekiq)
- Serialization/view hotspots (Jbuilder/serializer doing queries per record)
- Cache correctness (keys include versioning; avoid caching already-fast work)
- External HTTP: timeouts, retries, and failure modes
- Frontend: blocking assets, Turbo/Stimulus regressions, heavy DOM work

### Tests
- Coverage matches risk (migrations, payments, permissions => higher bar)
- Flaky patterns (time, randomness, background jobs, external calls)
- Request specs for endpoints; avoid overspecifying implementation

### DX / Ops
- Logs are useful, not noisy; include correlation IDs when available
- Feature flags: safe defaults, cleanup plan
- Observability: metrics/events where warranted for risky changes

## What NOT to do
- Don’t restate the diff back to the user.
- Don’t bikeshed style if RuboCop/Prettier would handle it.
- Don’t propose abstractions for one-off code.
- Don’t drown the user in nits if there are blockers.
- Don’t claim something is “slow” without pointing to a likely mechanism or asking for evidence (query plan, benchmark, prod metric).

## Output format (strict)

## Code Review — <scope>

### 🔴 Blocking
- **path/to/file.rb:123** — <issue>. <why it matters>.
  Suggestion: <concrete change / snippet>

### 🟡 Should fix
- ...

### 🟢 Nits
- ...

### Tests
- <what’s missing / what looks good>
- <commands to run, if relevant>

### Verdict
<Approve / Request changes / Needs discussion> — <one sentence why>

Omit empty sections.

## Stop conditions
- Diff > 1000 lines: ask to scope down, or do a sampling-based review and explicitly say it’s sampling-based.
- Generated/vendored files: skip and note them.
- Missing diff/base: ask once for PR number/link or base branch, then proceed when provided.
- Performance claims need evidence — flag suspected hot paths, but don't assert "this is slow" without a query plan, benchmark, or production signal to point at.

