---
name: refactor
description: Use when the user asks to refactor, restructure, clean up, simplify, extract, decompose, or "make this better" — without changing behavior. Also use when the user says "this is messy", "too complex", or "hard to test". Produces a safe, staged refactoring plan following Kent Beck / Sandi Metz principles.
---

# Refactor

Restructure code without changing behavior. Make the change easy, then make the easy change.

## Principles

1. Passes the tests
2. Reveals intention
3. No duplication
4. Fewest elements

If the refactor violates any of these, stop and reconsider.

## Before writing any code

1. **Verify tests exist.** If the code under refactor has no test coverage:
   - Write characterization tests first (tests that lock current behavior).
   - Get them green. Commit.
   - Then refactor.

2. **State the smell.** Name what's wrong:
   - Too many responsibilities (God object, fat controller)
   - Shotgun surgery (one change touches 10 files)
   - Feature envy (method uses another object's data more than its own)
   - Long method / deep nesting
   - Primitive obsession (passing hashes/strings where a value object fits)
   - Duplicated logic across files
   - Hard to test (too many dependencies, hidden side effects)

3. **Propose the plan.** List the steps as separate, individually-shippable changes:

Step 1: Extract X into Y (new file, no behavior change)
Step 2: Replace callers of X with Y
Step 3: Remove X
   Each step must pass all tests independently.

## Patterns to reach for (in this codebase)

| Smell | Pattern | Where it lives |
|---|---|---|
| Fat model | Extract service | `app/services/` |
| Complex query | Extract query object | `app/queries/` |
| Data access leaking locale/user context | Extract repository | `app/repositories/` |
| Reusable view logic | Extract ViewComponent | `app/components/` |
| Shared behavior across models | Extract concern (only if truly shared, not speculative) | `app/models/concerns/` |
| Business logic in callback | Extract to explicit service call | `app/services/` |
| Complex conditional | Replace with polymorphism or strategy | Depends on context |

Don't introduce a pattern that doesn't exist in the codebase without discussing it first.

## Execution rules

- **One thing per commit.** Rename in one commit, extract in another, delete in another.
- **No behavior changes.** If you spot a bug during refactoring, note it and fix it in a separate PR.
- **No formatting changes** on code you're not refactoring. Keep diffs minimal.
- **Run tests after every step:**
```bash
  bin/rspec spec/path/to/relevant_spec.rb

bash
bin/rake lint:fix
   Run linting after every step:
   ```bash
   bin/rake lint:fix
   ```

   Don't add dependencies without approval.

## Output format

### Refactor — <target (class/method/area)>

#### Smell
<what's wrong and why it matters>

#### Plan
1. <step> — <what changes, what stays>
2. ...

bin/rake lint
#### Step 1
<diff>

#### Step 2
<diff>

...

#### Verification
```bash
bin/rspec <relevant specs>
bin/rake lint
```

#### Risks / caveats
- <anything the reviewer should watch for>

**What NOT to do**

- Don't rename files and change logic in the same commit.
- Don't extract a class used in exactly one place (unless it's clearly a different responsibility).
- Don't introduce abstractions for hypothetical future use (YAGNI).
- Don't "improve" code you weren't asked to touch.
- Don't rewrite, refactor. Small steps, always green.
