---
name: performance-review
description: Use when the user asks for a performance review, perf audit, "is this slow?", "optimize this", "N+1", "query performance", "page is slow", "memory issues", or wants to review a change specifically for performance impact. Focused on Rails + Postgres + Heroku + Sidekiq stack.
---

# Performance Review

Evaluate code for performance problems on a Rails/Postgres/Heroku/Sidekiq stack. Evidence over intuition.

## When to use

- Reviewing a diff/PR specifically for perf impact
- Investigating a slow endpoint, job, or page
- Proactive audit of a hot path

## How to run the review

1. **Identify the hot path.** What code runs per-request, per-job, or per-record? Start there.

2. **Check queries first — they're almost always the bottleneck.**

   | Problem | What to look for |
   |---|---|
   | N+1 | `.each` / `.map` over association without `includes` / `preload` / `eager_load` |
   | Missing index | `where`, `order`, `find_by`, `joins` on unindexed columns; check `db/schema.rb` |
   | Unbounded load | `Model.all`, `where(...).each` on large tables — need `find_each` or pagination |
   | Expensive count | `count` on large unindexed scope; consider `counter_cache` |
   | Over-select | `SELECT *` when only a few columns needed; use `select` or `pluck` |
   | Query in view | SQL triggered inside ERB / ViewComponent — should be preloaded in controller |

3. **Check application-level patterns:**
   - Synchronous external API calls in request cycle → move to Sidekiq
   - Large object allocation in loops (string building, array concat)
   - Serializer/Jbuilder doing per-record queries or computation
   - Missing or broken caching (stale keys, caching cheap operations, no expiry)
   - File/image processing in the request thread

4. **Check background jobs:**
   - Jobs that touch the DB in bulk without `find_each` / batching
   - Missing `ActiveJob.perform_later` timeouts
   - Jobs that aren't idempotent (retries cause duplicate work)
   - Queue saturation from low-priority jobs blocking critical ones

5. **Check frontend (if applicable):**
   - Blocking JS in `<head>`, undeferred Stimulus controllers
   - Turbo Frames loading eagerly when they should be lazy
   - Large inline payloads (JSON blobs in data attributes)
   - Unoptimized images, missing lazy loading

6. **Check Heroku/infra signals (if available):**
   - Dyno queue time > 50ms p95 → scale out or speed up
   - Memory > 512MB (R14) → look for object bloat, leaks
   - Postgres connection saturation → check pool size, long transactions

## Output Format

**Performance Review —** `<scope>`

🔴 **Critical (measurable user/system impact)**
```
file:line — <problem>. <evidence or mechanism>.
Fix: <concrete suggestion>
```

🟡 **Likely problem (needs measurement)**
```
...
```

🟢 **Opportunity (low urgency)**
```
...
```

**Suggested measurements**
```
<specific query to EXPLAIN ANALYZE, endpoint to benchmark, metric to check>
```

**Verdict**

<OK for now / Needs fixes before merge / Needs load testing>

## Rules

- Don't say "this is slow" without pointing to a mechanism (query plan, object count, missing index).
- Don't optimize code that runs once at boot or in a rake task unless asked.
- Don't suggest caching as a first resort — fix the underlying problem first.
- If you're guessing, say "likely" and suggest how to measure.
- Reference `db/schema.rb` to verify indexes exist before flagging missing ones.

