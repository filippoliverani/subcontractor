---
name: pr-description
description: Use when the user asks to suggest, write, draft, or improve a PR description, prepare a PR for review, or says "open a PR" / "create a PR" / "PR description". Reads the project PR template and the current diff to produce a complete, ready-to-paste pull request body.
---

# PR Description

Generate a pull request description from the current diff using the project's PR template.

## Steps

1. **Read the PR template**
	```bash
	cat ./.github/pull_request_template.md
	```
	Follow its structure exactly. Do not invent sections or skip any.

2. **Get the diff**
	- If a PR already exists: `gh pr diff <number>`
	- Otherwise: `git diff main...HEAD`
	- If the user names a branch: `git diff main...<branch>`

3. **Understand intent**
	- Read commit messages: `git log main..HEAD --oneline`
	- If a Jira ticket is mentioned (FWMD-xxx), reference it.
	- If intent is still unclear, ask one question — then proceed.

4. **Fill every section of the template**
	- **Title:** `[FWMD-xxx]` Imperative summary (≤72 chars).
	- **Summary / What:** 2–4 sentences. What changed and why. Link Jira ticket.
	- **How:** Brief technical approach — key files, patterns used, trade-offs made.
	- **Rollback / risks:** Migrations, feature flags, external dependencies, data changes.
	- **Checklist:** Fill in the template's checklist items.

5. **Output the complete PR body in a single fenced Markdown block, ready to copy-paste.**

## Rules

Match the tone of the AGENTS.md: concise, direct, no filler.

- Never fabricate test results or screenshots.
- If the diff touches migrations, call out rollback safety explicitly.
- If the diff is empty or can't be obtained, say so and stop.
- Keep the description under 500 words — reviewers skim.
