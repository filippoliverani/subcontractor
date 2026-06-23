---
name: coding
description: >
  Execute a planned code change safely and efficiently. Use when the user has an implementation plan,
  a change plan, diff spec, or step-by-step implementation guide and wants the
  agent to apply it, or any time a build agent is writing code. Also triggers on: "execute the plan", "implement it", "implement this
  change", "apply these changes", "run the plan", "ship it", "make the changes",
  "implement the refactor".
---

# Coding

Apply a change plan produced by the Code Change Planner (or any structured diff spec) with maximum efficiency and zero unnecessary edits.

Never push a change unless explicitly instructed to.

## General Principles

Always apply Kent Beck's rules of simple design when planning or applying any code change:
1. Passes the tests
2. Reveals intention
3. No duplication
4. Fewest elements

Don't ever change the formatting of existing files unless explicitly requested by the user.
I want code diffs to be as small as possible to double-check them quickly.


## Code Style
- No dead code, don't leave commented-out code or unused methods.
- Keep methods short. If a method needs a comment to explain sections, it should be multiple methods.
- Prefer early returns over nested conditionals.
- Think about edge cases, error states, and nil/null/empty scenarios before coding.

## Error Handling & Robustness

- Handle errors explicitly. Don't swallow exceptions or use bare rescue/catch blocks.
- Fail fast and loud in development; fail gracefully in production paths.
- Validate inputs at system boundaries (controllers, API endpoints, public interfaces). Trust data inside the system.
- Never silently return nil/null when the caller expects a value. Raise, return a result object, or document the contract.

## Testing

- Test behavior, not implementation. Tests should survive refactoring.
- Write the test first when the requirement is clear. Write it alongside when exploring.
- One assertion per test when possible. Name the test after the behavior it verifies.
- Don't test private methods directly. Test through the public interface.
- Don't mock what you don't own. Wrap external dependencies and mock the wrapper.

## Security & Data Safety

- Never log, print, or expose secrets, tokens, passwords, or PII.
- Never hardcode credentials, API keys, or environment-specific values. Use environment variables.
- Sanitize and validate all external input (user input, API responses, file contents).
- Use parameterized queries. Never interpolate user input into SQL or shell commands.

## Git & Change Hygiene

- Each change should do one thing. If you need to refactor to enable a feature, that's a separate step.
- Write commit messages that explain why, not what. The diff shows the what.
- If a file is renamed or moved, do it in a dedicated step with no other changes.

## What NOT to Do

- Don't use magic values or magic strings. Extract to named constants.
- Don't suppress linter warnings without explaining why.
- Don't generate placeholder or stub implementations without clearly marking them as incomplete.
- Don't "improve" working code you weren't asked to touch.

## Execution Rules

Load ./AGENTS.md and use the test, lint and other verification commands defined there when needed.
Load the rails skill if working in a Rails project.

### Before Writing Any Code

1. **Read the full plan first.** Do not start editing after reading only the first step.
2. **Identify the invariant** — the behavior or API that must not change. Keep it visible throughout.
3. **Locate every file** referenced in the plan. Confirm they exist and match the expected current state. If a file differs from what the plan assumes, **stop and report the mismatch** before proceeding.

### Editing Strategy

- **One step at a time.** Follow the plan's sequence exactly. Do not reorder, skip, or combine steps.
- **Minimal touch.** Edit only the lines the plan specifies. Do not reformat surrounding code, fix linting warnings, rename unrelated variables, or add comments the plan didn't ask for.
- **Preserve whitespace conventions.** Match the file's existing indentation style (tabs vs spaces, width). Match the file's existing blank-line patterns between methods/blocks.
- **Preserve import order.** Add new imports in the position that matches the file's existing convention (alphabetical, grouped by source, etc.).
- **Copy identifiers exactly.** Variable names, method names, class names, string literals — use the exact casing and spelling from the plan. If the plan contains a typo, flag it instead of silently correcting.

### Add → Migrate → Remove

Follow this order strictly:

1. **Add** new code (functions, modules, classes). Nothing references it yet — the codebase must still work identically.
2. **Migrate** each caller one at a time. After each caller migration, the codebase must compile/pass tests.
3. **Remove** old code only after every caller has been migrated.
4. **Never** delete a function/class and recreate it in the same step. Always bridge through an intermediate state.

### After Each Step

- Verify the step's success criterion (test name, assertion, or manual check listed in the plan).
- If a test fails or the build breaks, **stop**. Report what failed and which step caused it. Do not attempt to fix forward without explicit approval.

### After All Steps

- Run the full test suite (or the subset specified in the plan).
- Confirm the pre-flight checklist from the plan is satisfied.
- Report: steps completed, tests passing, any deviations or surprises.

## Handling Ambiguity


| Situation                                          | Action                                                      |
| -------------------------------------------------- | ----------------------------------------------------------- |
| Plan says MODIFY but doesn't show a diff sketch    | Ask for clarification before editing                        |
| Plan references a file/method that doesn't exist   | Stop and report — do not create it unless the plan says ADD |
| Plan's diff sketch conflicts with the current code | Stop and report the conflict with both versions             |
| Multiple valid ways to implement a step            | Choose the smallest diff. If genuinely tied, ask            |
| Plan omits test updates but behavior changed       | Flag it — suggest a test spec but don't write tests unasked |


## Efficiency Guidelines

- **Batch reads.** Open all files needed for a step before making edits — avoid re-reading files mid-step.
- **Surgical edits.** Use targeted find-and-replace or line-range edits, not full-file rewrites. This preserves git blame and reduces merge conflict surface.
- **No roundtrips for confirmation** unless the plan is ambiguous or the code doesn't match expectations. Execute decisively.
- **Group related file edits** within a single step when the plan allows, to minimize tool calls.

## Output Format

After execution, report:

```markdown
# Execution Report

**Plan**: <plan title>
**Steps completed**: N/N
**Tests**: <pass/fail summary>

## Changes Made
| File | Action | Lines changed |
|------|--------|---------------|
| ...  | ...    | ...           |

## Deviations
- <any difference from the plan, with justification>

## Issues
- <any problems encountered, or "None">
```

## Principles

- **The plan is the spec.** Do not improve, extend, or second-guess it during execution. If something seems wrong, raise it — don't silently fix it.
- **Compilable at every commit.** The codebase must be valid after every step, not just at the end.
- **Traceability.** Every edit must map back to a specific step and change spec in the plan.
- **Fail fast, fail loud.** A stopped execution with a clear error report is better than a completed execution with hidden breakage.

