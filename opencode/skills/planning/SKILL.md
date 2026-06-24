---
name: planning
description: >
  Plan safe, minimal code changes before implementation.
  Use when the user wants to plan a new feature, a refactor, review a proposed change for side effects, prepare a code modification strategy, break a large change into steps, or assess risk before editing code.
  Also triggers on: "Plan how", "suggest how to", "plan this/a change", "what's the safest/best way to", "break this into steps", "change plan", "implementation plan", "refactor plan".
---

# Planning

Produce a precise, unambiguous change plan that minimizes risk and maximizes the chance of a clean first-pass implementation.

## General Principles

- Read before writing. Understand the existing code style and patterns before planning changes.
- Plan the smallest change that solves the problem.
- Make the change easy with pre-factoring and tidying up, then make the easy change.
- Don't reduce the degree of optionality without asking.
- Don't plan to refactor unrelated code in the same change.
- Consider what could break, not just what should work.
- When multiple approaches exist, briefly state the trade-offs and pick one. Don't silently choose.
- Match the abstraction level of surrounding code.
- Follow the conventions already established in the codebase. Search for similar solutions and replicate the style used there.

## Command-Line Tooling

Use modern, performant CLI tools for all code exploration during planning. They are faster, give better signal, and reduce noise. If a tool is missing, ask the user to install it (they can do so beforehand) rather than falling back silently to a slower legacy tool.

| Task | Use | Instead of | Why |
|------|-----|-----------|-----|
| Search file contents | `rg` (ripgrep) | `grep`, `ack` | Faster, respects `.gitignore`, multiline, type filters |
| Find files by name/path | `fd` | `find` | Simpler syntax, fast, gitignore-aware |
| View a file | `bat` | `cat` | Syntax highlighting, line numbers, git gutter |
| Structural / AST search | `ast-grep` (`sg`) | `rg` for code shape | Matches syntax, not text — precise for refactors |
| Interactive filtering | `fzf` | manual scanning | Fuzzy-select files, symbols, call sites |
| JSON inspection | `jq` | manual parsing | Query configs, lockfiles, API fixtures |
| YAML/TOML inspection | `yq` | manual parsing | Query CI config, `.rubocop.yml`, manifests |
| Directory overview | `eza --tree` or `lsd` | `ls -R` | Fast, readable tree with gitignore awareness |
| Line/diff stats | `tokei`, `git diff --stat` | `wc -l` | Per-language LOC, blast-radius sizing |
| Diff viewing | `delta` | `git diff` raw | Readable side-by-side, syntax-highlighted diffs |

### Tooling conventions

- Scope the blast radius with `rg` first; quantify it with `rg -l <pattern> | wc -l` or `tokei`.
- Use `ast-grep` when the change is structural (rename a method, change a call signature, swap a pattern) — it won't match comments or strings by accident the way `rg` can.
- Use `fd` to enumerate affected files by path/extension, then pipe into `bat`/`delta` for review.
- Prefer `rg --type ruby` (and similar type filters) over manual glob lists.
- Combine: `rg -l "OldClass" | fzf` to interactively confirm call sites before listing them in the plan.
- Never run destructive commands during planning. Exploration tools are read-only by design — keep it that way.

## Performance Awareness

- Don't optimize without evidence, but don't introduce obvious inefficiencies either.
- Watch for N+1 queries, unbounded loops, loading entire datasets into memory, and missing database indexes on new columns.
- If a change could impact performance at scale, flag it.

## Style

- Prefer clarity over cleverness.
- No premature optimization.
- Naming matters more than structure. Spend time on variable, method, and class names that read like prose.
- Keep the happy path unindented; handle edge cases and errors first.

## Dependencies & External Code

- Don't add gems, packages, or libraries without explicit approval.
- Don't add middleware, initializers, or framework plugins without explicit approval.
- When a dependency is proposed, state why the standard library or existing dependencies can't solve it.

## What NOT to Do

- Don't introduce patterns that don't already exist without discussion.

## Planning Workflow

### 1. Understand the Goal

- Restate the change in one sentence: what changes, and why.
- Identify the invariant: what must NOT change (public API, behavior, output format, test expectations).

### 2. Map the Blast Radius

Find every call site with `rg` (or `ast-grep` for structural matches), enumerate affected files with `fd`, and size the result with `tokei` or `rg -l <pattern> | wc -l`. List every file, class, method, and call site affected. Classify each as:

| Tag | Meaning |
|-----|---------|
| MODIFY | Logic changes inside this unit |
| ADAPT | Signature/type/import changes only — no logic change |
| VERIFY | Not changed, but could break; needs test/review |

If blast radius > 5 files, split into sequential sub-plans (each independently shippable).

### 3. Sequence the Changes

Order so the codebase compiles/passes tests at every intermediate step:

1. Add new code (functions, classes, modules) — nothing calls it yet.
2. Migrate callers to the new code, one at a time.
3. Remove old code only after all callers are migrated.
4. Update tests last (or in lockstep with each caller migration).

Never delete and rewrite simultaneously — always add → migrate → remove.

### 4. Identify Edge Cases and Risks

For each MODIFY unit, answer:

- What inputs/states exercise the changed path?
- Are there implicit contracts (nil returns, exception types, ordering guarantees, encoding assumptions)?
- Could memoization, caching, or lazy initialization see stale state after the change?
- Are there concurrency, thread-safety, or re-entrancy concerns?

### 5. Write the Change Spec

One spec per MODIFY/ADAPT unit:

```
## File: <path>

### <method/class/function>
- **Action**: MODIFY | ADAPT | ADD | REMOVE
- **Current behavior**: <one line>
- **New behavior**: <one line, or "unchanged" for ADAPT>
- **Key constraint**: <invariant that must hold>
- **Diff sketch** (pseudocode, not final code):
  - Remove: <what goes away>
  - Add: <what replaces it>
- **Risk**: LOW | MEDIUM | HIGH — <why>
- **Verify**: <how to confirm correctness — test name, manual check, assertion>
```

### 6. Pre-flight Checklist

Before handing to the coding agent, confirm:

- [ ] No public API signature changes (unless explicitly intended)
- [ ] No behavior changes beyond stated goal
- [ ] Every intermediate step is valid (compiles, tests pass)
- [ ] Edge cases from Step 4 have corresponding VERIFY entries
- [ ] Rollback path exists (can revert each sub-plan independently)

## Output Format

```
# Change Plan: <title>

Goal: <one sentence>
Invariant: <what must not change>
Blast radius: <N files: list>

## Sequence

### Step 1: <description>
<change specs for this step>

### Step 2: <description>
<change specs for this step>

...

## Risks & Mitigations
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| ...  | ...       | ...    | ...        |

## Pre-flight
- [ ] checklist items
```

## Principles

- Minimal diff: change the fewest lines possible. If a refactor isn't needed for the goal, don't include it.
- No drive-by fixes: resist the temptation to "clean up" unrelated code. Each plan has one goal.
- Byte-identical behavior: unless the goal explicitly changes behavior, the output of every public method must be identical before and after for all inputs. Call out any edge case where this isn't guaranteed.
- Assume the coding agent is literal: it will do exactly what the spec says. Ambiguity → bugs. Be precise about what to add, remove, and keep.
- Name the tests: reference specific test files/cases that validate each change. If no test exists, flag it and include a test spec.

