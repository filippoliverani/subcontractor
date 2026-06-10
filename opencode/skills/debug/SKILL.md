---
name: debug
description: Use when the user reports a production bug, error, exception, Datadog error, unexpected behavior, "this is broken", "users are seeing X", or asks to investigate / debug / troubleshoot an issue. Structured incident debugging for Rails + Heroku + Datadog stack.
---

# Debug

Systematic debugging for production and staging issues. Narrow scope before writing code.

## Inputs (ask if missing)

- **What's happening?** Error message, Datadog link, user report, or screenshot.
- **Where?** Endpoint, page, job, etc..
- **Since when?** Timestamp, deploy, or PR that might correlate.
- **Who's affected?** All users, specific role, specific product/region.

## Process

### 1. Gather evidence (before touching code)

- **Datadog:** search framework-marketplace project on Datadog for the exception and for logs.
Note: frequency, affected users, stack trace, breadcrumbs, tags.
- **Recent deploys:** `gh pr list --state merged --limit 10` — correlate timing.
- **Reproduce:** try to reproduce locally. If you can't, say so.

### 2. Form a hypothesis

State it explicitly:
> "I believe X happens because Y, which was introduced by Z."

If you have multiple hypotheses, rank by likelihood and testability.

### 3. Verify

- Read the suspected code path end to end.
- Check for: nil propagation, time/timezone bugs, race conditions, missing authorization, changed API contracts, migration side effects.
- If the bug is in a Sidekiq job: check idempotency, retry behavior, argument serialization.
- If the bug is in Solidus: check overrides in `app/overrides/`, subscribers in `app/subscribers/`, and extensions in `app/extensions/`.

### 4. Fix

- **Minimal fix only.** Don't refactor in the same change.
- Write a regression test that fails before the fix and passes after.
- If the fix is a hotfix: note deploy urgency and rollback plan.

### 5. Deliver

## Output Format

**Debug —** `<issue summary>`

**Evidence**
```
Error: <link or event ID>
Frequency: <X times in Y hours>
Affected: <scope>
```

**Hypothesis**

<1–3 sentences>

**Root cause**

<what's actually wrong, with file:line>

**Fix**
<minimal diff>

**Regression test**
<spec that covers the bug>

**Verification**
```
bin/rspec spec/path/to/file_spec.rb:42
```

**Follow-up**
```
<Jira ticket for deeper fix if this is a band-aid>
<Monitoring to add>
```


## Rules

- Don't guess at the fix before reading the stack trace and code path.
- Don't conflate symptoms with root cause.
- Don't expand scope — fix the bug, not the neighborhood.
- If you can't reproduce it, say so and propose instrumentation (logs, Datadog breadcrumbs, metrics) instead of a speculative fix.
- Always include a regression test.
- Reference Jira (FWMD-xxx) if a ticket exists.
