---
name: git-commit
description: Generate high-quality git commits with context-awareness, and multi-commit support. Use when the user says "commit", "create commit", "amend commit", "write commit message"
---

# Git Commit

Generate well-structured git commits following the seven rules of great commit messages. This skill handles single commits, multi-commit plans, and autonomous workflows. Of the seven rules, Rule 7 — **use the body to explain what and why, not how** — is the most important to internalize.

## Commit Message Rules

### 1. Separate subject from body with a blank line

Git tools (`log`, `shortlog`, `rebase`) depend on this separation. Simple changes (typo fixes, self-explanatory updates) need only a subject. Add a body when the motivation isn't obvious from the code, when there are side effects to explain, or when the change requires context.

### 2. Aim for 50 characters in the subject, keep it under ~72

50 characters is the target — it forces concise thinking. Around 72 characters is a commonly recommended maximum for readability; many tools and UIs wrap or truncate beyond this. If you struggle to summarize, the commit may be doing too many things (see Workflows for multi-commit planning).

### 3. Capitalize the subject line

### 4. Do not include ticket numbers in the subject line

### 5. No period at the end of the subject line

### 6. Use the imperative mood in the subject line

The subject should complete: "If applied, this commit will **[your subject line]**"

- "If applied, this commit will **refactor subsystem X for readability**" — correct
- "If applied, this commit will **fixed bug with Y**" — wrong (past tense)
- "If applied, this commit will **more fixes for broken stuff**" — wrong (no verb)

Imperative mood is required only in the subject. The body can be conversational.

### 7. Wrap the body at 72 characters

Git doesn't wrap automatically. 72 characters leaves room for indentation while staying under 80 columns.

### 8. Use the body to explain what and why, not how

This is the most important rule. The diff shows WHAT changed and the code shows HOW. The commit message's job is to explain WHY.

Specifically explain:

- What problem this commit solves and why the change was necessary
- Why this approach was chosen (if not obvious)
- Side effects, trade-offs, or unintuitive consequences
- What was wrong with the previous behavior

## Message Structure

The message body description should be short and in a format similar to
"Change/Fixes/(action) ... to/so that/(goal) ... becuase/why/(reason)"

### Subject only (simple changes)

```
Fix typo in user guide
```

### Subject + body (complex changes)

```
Fix session timeout on inactive tabs

Switches to tracking the last activity timestamp in localStorage so
all tabs share the same activity signal and refresh accordingly.

The auth token was being refreshed based on wall-clock time rather
than user activity. Tabs left open overnight would expire and force
a re-login on the next interaction, even if the user had been active
in other tabs.
```

### Writing style

Write commit message bodies as natural prose — like explaining a change to a colleague. Prefer flowing paragraphs that tell the story of why the change was needed. Bullet points are acceptable when listing genuinely helps clarity, but default to prose. Avoid "Files changed:" sections or documentation-style enumerations — the diff already shows what changed.

## Examples

These demonstrate the preferred style: natural prose focused on the WHY.

### Simple fix with concise body

```
Improve device service performance

Removes useless loops and adds memoization
```

### Bug fix with detailed explanation

```
Fix missing images in transactional emails

Includes protocol in action_mailer.asset_host configuration to generate
absolute URLs instead of protocol-relative URLs. This ensures all
images (logos, social icons, payment method icons) render correctly in
order confirmations, cancellations, and other transactional emails.

Previous configuration generated URLs like //staging.frame.work/assets/...
which failed to load in email clients. Now generates proper absolute
URLs like https://staging.frame.work/assets/... with protocol and
port included.
```

### Monitoring and observability

```
Add monitoring for device name pattern mismatches

Logs to Sentry when the regex doesn't match for devices
(laptops/desktops) so we're alerted if the naming format changes.

ProductPreview#name_without_family uses a regex to strip the processor
family suffix (e.g., "(AMD Ryzen™ AI 300 Series)") from device names. If
admins change the naming convention, this regex could silently fail,
leading to duplicated information in the UI.
```

### Refactoring with rationale

```
Consolidate duplicate validation logic into shared module

Three controllers were each implementing their own address validation
with subtle differences that caused inconsistent error messages. Moving
this into a shared module ensures consistent behavior and makes the
validation rules easier to update when we add new country formats.
```

### Dependency update with context

```
Upgrade Redis client from 4.x to 5.x

The 4.x series has a connection leak under high concurrency that
causes intermittent timeouts during peak traffic. Version 5.x
redesigns the connection pool to be fully thread-safe, which
eliminates the leak and also simplifies our retry configuration
since the built-in retry logic now covers our use cases.
```

### Security improvement

```
Enable SSL verification for SerialInfoClient

Removes ssl_verifyhost and ssl_verifypeer options that disabled
TLS certificate validation on the signing server connection.
Typhoeus defaults to verifying both host and peer, which
prevents Man-in-the-Middle attacks on serial info lookups.

Client certificate options (sslcert/sslkey) remain unchanged
for mutual TLS authentication.
```

## Workflows

Regardless of the scenario, follow these steps:

**1. Identify what to commit**

- If the user staged changes: verify staged changes exist (`git diff --cached`). If nothing is staged, stop and inform the user.
- If the agent generated changes: analyze all uncommitted changes (staged, unstaged, untracked files the agent created).

**2. Gather context**

Detect the base branch and fetch the latest state from the remote. Analyze the branch history (commits since divergence from base) to understand the overall narrative and how the new commit(s) fit in.

**3. Plan the commits**

For a single commit: draft the message directly.

For multiple commits: group changes into logical, atomic units. Each commit should address a single concern and have a clear, concise subject line. If a commit resists a concise subject, split it further.

**4. Draft messages**

Apply all seven rules from the Commit Message Rules section. Write in the style described in Writing Style. Use context from the branch history to inform the reasoning, but describe only what the specific commit does — not the entire branch.

If you cannot deduce the motivation with high confidence, ask the user (when present) rather than inventing reasoning. Good questions: What problem does this solve? Why was the previous approach insufficient? What's the business or user impact?

**5. Present and execute**

The approach depends on the scenario:

- **User-staged changes:** The user has staged files and asked for a commit. Show the proposed message in a `gitcommit` block and wait for approval. Commit only after the user confirms. The user may iterate on the message before accepting.

- **Agent-generated changes with a user present:** The agent has produced changes and needs to commit its work. Present the full commit plan (all grouped commits with their messages). The user can adjust groupings, refine messages, merge or split commits. Use your judgment on presentation — for 2–3 straightforward commits, show all at once; for complex plans, walk through them one at a time. Execute sequentially only after approval.

- **Autonomous (no user interaction):** The agent is running in a CI/cloud environment with no user to interact with. Proceed directly to execution without a confirmation step. Follow the same rules and quality standards — the only difference is no pause for human review.

For multiple commits, always execute sequentially in logical order.

## Context Awareness

### Repository-specific conventions

The skill follows its built-in rules by default. If the repository provides additional guidance via AGENTS.md, .gitmessage, or similar configuration files, those conventions take precedence — even when they conflict with the rules here (e.g., a repo using Conventional Commits with lowercase subjects). Check for these files when operating in a new repository.

## Output Format

Present commit messages wrapped in triple backticks with `gitcommit` as the language identifier:

```gitcommit
Your commit message here
```

When presenting a multi-commit plan, use one `gitcommit` block per commit, with a brief note before each indicating which files it covers.

