---
name: heroku
description: Read and interpret Heroku app health through a Heroku MCP server. Use when the user asks about dyno throughput, dyno memory usage (RSS, R14/R15), request latency (p95/p99), Ruby/Rails runtime metrics, Sidekiq/worker health, scaling decisions, or incident triage on a Heroku app. Triggers on phrases like "is my app memory-bound", "what's the throughput", "should I scale up or out", "why is staging slow", "check dyno metrics", "R14", "H12".
---

# Heroku

Read Heroku dyno metrics and Ruby/Rails runtime data through a Heroku MCP server, then advise on capacity, code, and config — not just report numbers.

## Core concepts

| Term | Meaning |
|---|---|
| Dyno | Isolated container running one process (web, worker, etc.). |
| Formation | Dyno types, sizes, and quantities for an app. |
| Throughput | Requests/sec served by web dynos (router metric). |
| R14 / R15 | Memory quota exceeded (R14 = swapping, R15 = killed). Top Rails signal. |
| H12 | Request timeout — router killed a request at 30s. |
| RSS | Resident memory per dyno; compare to the dyno size limit. |

Metrics (memory, throughput, latency, dyno load) exist only on **Standard and Performance** dynos. Eco/Basic emit logs only. If needed check Dynos specs on Heroku docs https://devcenter.heroku.com/articles/dyno-sizes


## Workflow

0. **Fail early if requirements not met**
   - Stop with an error message if Heroku MCP is not enabled
1. **Discover tools first.** Do not assume MCP tool names; enumerate the connected server's capabilities and map them to the intents below.
2. **Confirm target.** State app name and environment (staging vs. production) before acting.
3. **Read the relevant metric** for the requested window.
4. **Cross-check logs** for R14/R15/H12 — ground truth over gauges.
5. **Produce the output format** below.

## Intent to action

- **Dyno throughput** — Read router throughput for web. Report req/s, total requests, status breakdown (2xx/4xx/5xx), and latency p50/p95/p99. Rising p95 with flat throughput = saturation.
- **Dyno memory** — Read RSS per dyno vs. its size limit. Flag any dyno above ~80%. Confirm with R14/R15 log events.
- **Ruby/Rails runtime** — If an APM add-on is connected, read request queue time, GC time/allocations, per-endpoint latency. Otherwise infer from memory-growth shape, throughput-vs-latency curve, and R14 frequency.
- **Sidekiq/workers** — Read worker dyno load/memory separately from web. If Redis add-on info is available, check queue latency and connected clients.

## Decision heuristics

- Memory climbs steadily, never plateaus -> leak/fragmentation. Recommend jemalloc (`MALLOC_ARENA_MAX=2` or the jemalloc buildpack) before scaling up.
- R14 under load, flat when idle -> undersized dyno or too many Puma workers. Check `WEB_CONCURRENCY` (~200-300MB per worker for typical Rails).
- High p95, low CPU/memory -> downstream bottleneck (DB, external API, N+1). Fix the query/add caching; don't scale dynos.
- Throughput capped, request queue time rising -> scale out (more web dynos), not up.
- H12 timeouts -> a slow endpoint. Move slow work to Sidekiq.

## Time windows

Default to last 1h for "right now", last 24h for trends, last 7d for capacity planning. Always state the window used.

## Safety

- Never print config var values (they contain `DATABASE_URL`, API keys, `ELASTICSEARCH_URL` with embedded passwords). Reference names only.
- Restarting, scaling, or setting config is destructive — confirm with the user before any write action. Read actions need no confirmation.
- Never act on production without an explicit go-ahead.
- All Heroku timestamps and Scheduler entries are UTC — convert when reporting.

## Output format

1. **Verdict** — one line: healthy / degraded / at-risk, and why.
2. **Numbers** — throughput, p95, memory %, error rate, R14 count in a compact table.
3. **Hypothesis** — code, config, or capacity.
4. **Action** — concrete and minimal (e.g. "set `WEB_CONCURRENCY=2`").
5. **Window/source** — time range and metric source used.

## Example

> **User:** Is staging memory-bound?
>
> Discover metrics tool -> read web RSS + R14 events, last 1h -> RSS 470MB/512MB (92%), 14 R14 events.
>
> **Verdict:** At-risk — staging web dynos are swapping.
> **Numbers:** RSS 470/512MB (92%), 14x R14/hr, throughput 3 req/s, p95 1.2s.
> **Hypothesis:** Too many Puma workers for a Standard-1X.
> **Action:** Set `WEB_CONCURRENCY=2` or upgrade to Standard-2X. Confirm before applying.
> **Window/source:** Last 60 min, Heroku native metrics.

