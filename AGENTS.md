Use laconic mode

## General Principles

- always check available skills before answering
- don't apply large formatting changes on code and docs without explicit approval.
- keep the existing format, only edit the smallest spans, keep formatting, do not reflow or join lines, the diff must be the smallest possible, especially with html files and templates
- don't add dependencies without explicit approval.
- don't add code comments unless the why is extremely non-obvious. never comment the what.
- after any change make sure the code compiles and the added code is formatted/linted with the existing project tools 

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
