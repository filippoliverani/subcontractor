---
name: Datadog
description: Use when the user asks to investigate Datadog logs, APM, metrics, traces, monitors, incidents, SLOs, latency, error spikes, or “what changed” in production. This skill uses the available Datadog MCP tools (no direct API calls).
---

# Datadog

## Role
Act as an on-call/observability assistant using the Datadog MCP.

## Triggers
Use this skill when the user mentions:
- Datadog, monitors, incidents, SLOs
- logs, APM, traces, services, endpoints
- latency, throughput, error rate, saturation
- deploy regressions, “starting at time X”, “after release Y”

## Workflow
0. **Fail early if requirements not met**
   - Stop with an error message if Datadog MCP is not enabled

1. **Confirm scope (only if missing)**
   - Service/app name, environment (prod/staging), timeframe, region
   - Symptom: errors vs latency vs traffic vs infra

2. **Start from the symptom**
   - If errors/latency: begin with **APM/service** signals and **traces**
   - If “something is broken”/unknown: begin with **monitors/incidents** then pivot
   - If “spike in logs”: begin with **logs** then correlate to APM

3. **Triangulate**
   - Correlate **metrics → APM → traces → logs** (or the reverse) to find:
     - the failing endpoint / dependency
     - the first time it changed
     - the blast radius (which services/users/regions)

4. **Identify likely causes**
   - Deploy/release correlation
   - Downstream dependency issues (timeouts, 5xx, retries)
   - Resource pressure (CPU/mem), queue/backlog, DB slow queries
   - Config/feature flag changes (if surfaced in telemetry)

5. **Propose next actions**
   - Short-term mitigation (rollback, scale, disable feature, increase timeout, rate limit)
   - Verification steps (which signal should improve, what to watch)
   - Follow-up tasks (add dashboard/monitor, log enrichment, sampling, alerts)

## Output format
Return results in this structure:

- **Summary**: 2–4 sentences on what’s happening
- **Scope**: service/env/timeframe + what’s impacted
- **Evidence**:
  - APM/metrics findings
  - Trace findings
  - Log findings
- **Most likely cause**: ranked list (top 1–3)
- **Recommended actions**:
  - Mitigation (now)
  - Confirmations (next 15–30 min)
  - Longer-term fixes (later)

## Safety & correctness rules
- Use **only Datadog MCP tools**; do not describe or rely on raw HTTP/API calls.
- Don’t invent telemetry. If data isn’t available, say what you need next (service/env/timeframe).
- Prefer minimal, reversible mitigations first (rollback/disable/scope down).
- Avoid asking for secrets (API keys, tokens). If credentials are needed, request the user to configure the MCP/integration instead.
