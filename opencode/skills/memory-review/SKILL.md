---
name: memory-review
description: >-
  Reproduce, measure, and diagnose Rails memory growth and object
  retention with derailed_benchmarks. Use when investigating a memory leak,
  rising RSS/dyno memory, OOM/R14 errors, retained objects, allocation
  hotspots, or "why does this endpoint keep growing memory". Triggers:
  "review memory usage", "derailed", "perf:mem_over_time", "perf:objects", "memory leak", "retained
  objects", "RSS keeps growing", "memory bloat", "allocation report".
---

# Memory Review

Reproduce production-like Rails request memory growth locally, collect
allocation/retention data, and tie retained objects to a concrete fix. Work in
this order: confirm setup -> probe for growth -> probe allocations/retention ->
report with before/after evidence.

## 1. Setup

Before running any probe:

1. Make sure derailed_benchmarks is wired-up, check if benchmark rake tasks or similar are defined
2. Verify that the existing rake tasks boot in  `RAILS_ENV=production` so you measure production code
    paths (eager loading, no dev reloader, real middleware stack).
3. Use **dummy** env vars only. Never use real production credentials, hosts, or tokens.
4. Make sure assets are precomplied first if production rendering needs a compiled manifest

## 2. Memory Growth Probe (`perf:mem_over_time`)

Hit the endpoint repeatedly and watch RSS over time:

```bash
ie. TEST_COUNT=100 PATH_TO_HIT=https://example.com/target_path bin/rake benchmark:mem_over_time
```

Interpret the output:

- **Warmup rise that plateaus** -> acceptable. Caches and pools fill once, then flatten.
- **Monotonic growth across many samples** -> retained objects or allocator
  fragmentation. This is the signal to escalate to the retention probe.
- If the run finishes before a second sample prints, **raise `TEST_COUNT`** until
  you get a clear slope.

Record the **RSS slope** (start vs end, and whether it flattens). This is your baseline.

## 3. Allocation & Retention Probe (`perf:objects`)

Once growth is reproduced, find *what* is being retained:

```bash
ie. TEST_COUNT=100 PATH_TO_HIT=https://example.com/target_path bin/rake benchmark:objects
```

Read these sections first, in this order:

1. **`Total retained`** — the headline number. Track this before/after a fix.
2. **`retained objects by class`** — what kind of objects survive GC.
3. **`allocated memory by gem/file/location`** — where allocations originate
   (often points straight at the offending gem or app file).
4. **`Retained String Report`** — duplicated/un-frozen strings are a frequent culprit.

Use a smaller `TEST_COUNT` here (allocation tracing is expensive); 50 is usually enough.

## 4. Diagnosis Heuristics

- Retained `String`/`Hash`/`Array` growing with request count -> look for caching
  in constants, class variables, or memoization keyed on per-request data.
- A gem dominating `allocated memory by location` -> check for per-request object
  creation that should be reused (connections, serializers, parsers).
- RSS grows but `perf:objects` retained is flat -> suspect **allocator
  fragmentation**, not a leak (consider jemalloc / `MALLOC_ARENA_MAX`).
- Always A/B suspicious **monitoring/APM middleware** — disable it and re-run; it
  is a common hidden source of retention.

## 5. Report Checklist

A complete report includes:

- Exact endpoint and the full environment command shape used.
- Baseline RSS slope and post-change RSS slope.
- `perf:objects` retained totals **before and after** the change.
- The retained classes/files that led to the diagnosis.
- A/B results for any monitoring middleware tested.
- Test/format commands run.
- Generated directories (e.g. `tmp/derailed/`) added to `.gitignore`.

