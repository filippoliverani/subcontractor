---

name: test-triage
description: Use when the user shares a failing test, CI failure output, red spec, or asks to "fix this test" / "triage CI" / "why is this failing" / "flaky spec". Diagnoses test failures quickly and suggests minimal fixes.
---

# Test Triage

Diagnose a failing test and suggest the smallest fix. Do not shotgun-debug.

## Inputs

- A file + line: `spec/path/to/file_spec.rb:42`
- Or CI output (pasted or linked)
- Or just "CI is red" — in that case, run `bin/rspec` or check recent CI output first.

## Diagnosis steps

1. **Read the failure message and backtrace carefully.**
   Identify: expected vs actual, exception class, originating line.

2. **Classify the failure:**
   - 🐛 **Real bug** — code doesn't match spec's expectation, and the spec is correct.
   - 🧪 **Bad test** — spec is wrong, brittle, or over-specified.
   - 🎲 **Flake** — passes locally, fails in CI. Common causes:
     - Time-dependent (`Time.current`, `travel_to` missing)
     - Random ordering (`order(:id)` assumed but not guaranteed)
     - Async/Sidekiq (`perform_enqueued_jobs` missing)
     - Database state leaking between examples
     - Capybara timing (missing `have_content` waits, `sleep` instead of matchers)
   - 🔧 **Environment** — missing env var, service down, seed data.

3. **Reproduce locally first:**
   ```bash
   bin/rspec spec/path/to/file_spec.rb:42
   ```

   For JS tests:
   ```bash
   yarn vitest run spec/javascript/path/to/file.spec.js
   ```

   For system specs:
   ```bash
   CAPYBARA_JS_DRIVER=selenium_chrome bin/rspec spec/system/foo_spec.rb
   ```

   Read the test AND the code under test. Don't guess from the failure message alone.

   Propose the minimal fix. One of:
   - Fix the code (if the spec correctly describes desired behavior)
   - Fix the spec (if the spec is wrong or brittle)
   - Mark as flaky with a clear explanation + follow-up ticket

## Output format

### Test Triage — <file:line>

**Classification:** 🐛 Real bug / 🧪 Bad test / 🎲 Flake / 🔧 Environment

**Root cause:** <1–3 sentences>

**Fix:**
<minimal diff or code snippet>

**Verification:**
```bash
bin/rspec spec/path/to/file_spec.rb:42
```

**Related:** <Datadog link, Jira ticket, or "none">

## Rules

- Don't rewrite the entire test file. Fix the failing example.
- Don't add sleep to fix timing issues — use proper Capybara matchers.
- Don't stub what you don't own (per project conventions).
- If the failure is in CI only and can't be reproduced locally, say so and suggest seed/ordering investigation.
- Run linting after any fix: `bin/rake lint:fix`
