## General Principles

- Read before writing. Understand the existing code style and patterns before making changes.
- Make the smallest change that solves the problem.
- Make the change easy pre-facatorug and tidying up and then make the easychange.
- Don't reduce the degree of optionality without asking.
- Don't refactor unrelated code in the same change.
- Don't apply large formatting changes on code and docs without explicit approval.
- Don't use an edit tool that automatically reformats code, keep the existing format, the diff must be the smallest possible, especially with HTML and ERB files.
- Don't add dependencies without explicit approval.
- Don't add code comments unless the why is extremely non-obvious. Never comment the what.
- After any change Make sure the code compiles and it's formatted/linted with the existing project tools 

Always apply Kent Beck's rules of simple design when planning or applying any code change:
1. Passes the tests
2. Reveals intention
3. No duplication
4. Fewest elements

Don't ever change the formatting of existing files unless explicitly requested by the user.
I want code diffs to be as small as possible to double-check them quickly.

## Code Style

- Follow the conventions already established in the codebase. Search for similar solutions and replicate the style used there
- Match the abstraction level of surrounding code.
- Prefer clarity over cleverness.
- No premature optimization.
- No dead code, don't leave commented-out code or unused methods.
- Naming matters more than structure. Spend time on variable, method, and class names that read like prose.
- Keep methods short. If a method needs a comment to explain sections, it should be multiple methods.
- Prefer early returns over nested conditionals.
- Keep the happy path unindented; handle edge cases and errors first.

## Reasoning & Planning

- Before writing any code, briefly state your understanding of the problem and your approach.
- If the change touches more than one file, outline the plan before implementing.
- Think about edge cases, error states, and nil/null/empty scenarios before coding.
- Consider what could break — not just what should work.
- When multiple approaches exist, briefly state the trade-offs and pick one. Don't silently choose.

## Error Handling & Robustness

- Handle errors explicitly. Don't swallow exceptions or use bare rescue/catch blocks.
- Fail fast and loud in development; fail gracefully in production paths.
- Validate inputs at system boundaries (controllers, API endpoints, public interfaces). Trust data inside the system.
- Never silently return nil/null when the caller expects a value. Raise, return a result object, or document the contract.

## Testing

- Write the test first when the requirement is clear. Write it alongside when exploring.
- Test behavior, not implementation. Tests should survive refactoring.
- One assertion per test when possible. Name the test after the behavior it verifies.
- Don't test private methods directly. Test through the public interface.
- Don't mock what you don't own. Wrap external dependencies and mock the wrapper.

## Security & Data Safety

- Never log, print, or expose secrets, tokens, passwords, or PII.
- Never hardcode credentials, API keys, or environment-specific values. Use environment variables.
- Sanitize and validate all external input (user input, API responses, file contents).
- Use parameterized queries. Never interpolate user input into SQL or shell commands.

## Performance Awareness

- Don't optimize without evidence, but don't introduce obvious inefficiencies either.
- Watch for N+1 queries, unbounded loops, loading entire datasets into memory, and missing database indexes on new columns.
- If a change could impact performance at scale, flag it.

## Dependencies & External Code

- Don't add gems, packages, or libraries without explicit approval.
- Don't add middleware, initializers, or framework plugins without explicit approval.
- When a dependency is proposed, state why the standard library or existing dependencies can't solve it.
- Pin versions. No floating version constraints.

## Git & Change Hygiene

- Each change should do one thing. If you need to refactor to enable a feature, that's a separate step.
- Write commit messages that explain why, not what. The diff shows the what.
- If a file is renamed or moved, do it in a dedicated step with no other changes.

## What NOT to Do

- Don't introduce patterns that don't already exist without discussion.
- Don't write speculative code for future requirements that haven't been asked for (YAGNI).
- Don't use magic values or magic strings. Extract to named constants.
- Don't suppress linter warnings without explaining why.
- Don't generate placeholder or stub implementations without clearly marking them as incomplete.
- Don't "improve" working code you weren't asked to touch.

## When Unsure

- Ask. Don't guess at requirements.
- If a task is ambiguous, state your assumptions before writing code.
- If a change feels too big, propose splitting it.
- If you're uncertain about the existing codebase's convention for something, grep for examples before inventing a new pattern.
- If you can't find enough context to be confident, say so. A question is always better than a wrong assumption baked into code.

## Response Format

- Show only the files and lines you changed. Don't repeat unchanged code unless needed for context.
- When editing existing files, provide the minimal diff, not the entire file.
- After the code, briefly explain what you changed and why. Keep it to 2-3 sentences.
- If the change has any caveats, migration steps, or manual actions required, list them explicitly at the end.

## Additional Instructions

CRITICAL: When you encounter a file reference (e.g., @rules/general.md), use your Read tool to load it. They're relevant to the SPECIFIC task at hand.
