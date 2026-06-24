---
name: Git
description: Safe, productive git workflows for AI agents. Use when committing, amending commit messages, rebasing (interactive or onto a branch), squashing, merging, resolving conflicts, cherry-picking, stashing, tagging, force-pushing, or undoing changes. Triggers on "rebase", "amend commit", "reword commit", "squash", "fix git history", "resolve conflict", "force push".
---

# Git Operations

Operate git safely and deterministically. Never modify history without explicit user approval. Verify state before and after every operation.

## Always run first

Before any git operation:

```bash
git status && git log --oneline -5 && git branch --show-current
```

## Hard rules

- Never use bare `git rebase -i` — it opens an editor you cannot control. Always script it with `GIT_SEQUENCE_EDITOR`.
- Never `git push --force`. When force-pushing is approved, use `git push --force-with-lease`.
- Never delete a remote branch or tag without explicit approval.
- Never `git reset --hard` or discard uncommitted changes without approval.
- Never use `git checkout --ours/--theirs` on a whole file without asking.
- Prefer `git revert` over `git reset` for anything already pushed.
- If a rebase, merge, or cherry-pick goes wrong: `--abort` immediately. Do not recover manually.
- Always name stashes with `-m`.
- After any history-modifying operation, verify with `git log --oneline -10`.

## Commits

For authoring commit messages, load the `git-commit` skill.

```bash
# amend most recent commit, keep message, add staged files
git add <file> && git commit --amend --no-edit
```

## Interactive rebase (scripted only)

Find the target SHA first with `git log --oneline -10`, then:

```bash
# reword one commit
GIT_SEQUENCE_EDITOR="sed -i 's/^pick TARGET_SHA/reword TARGET_SHA/'" git rebase -i TARGET_SHA~1
git commit --amend -m "new message"
git rebase --continue

# reword multiple commits
GIT_SEQUENCE_EDITOR="sed -i 's/^pick SHA1/reword SHA1/; s/^pick SHA2/reword SHA2/'" git rebase -i SHA2~1
# at each stop: git commit --amend -m "..." && git rebase --continue

# squash last N into the first
GIT_SEQUENCE_EDITOR="sed -i '2,\$s/^pick/squash/'" git rebase -i HEAD~N

# drop a commit
GIT_SEQUENCE_EDITOR="sed -i 's/^pick TARGET_SHA/drop TARGET_SHA/'" git rebase -i TARGET_SHA~1
```

On any failure: `git rebase --abort`.

## Rebase onto a branch

```bash
git fetch origin && git rebase origin/main
# on conflict: resolve file, git add <file>, git rebase --continue
# if messy: git rebase --abort
```

## Merging

```bash
git merge feature-branch --no-ff
```

Conflicts: list with `git diff --name-only --diff-filter=U`, resolve each understanding both sides, `git add <file>`, then `git merge --continue`.

## Branching

```bash
git checkout -b feature/short-description
git branch -d branch-name                 # -D only if disposable
git push origin --delete branch-name      # ASK FIRST
```

## Cherry-pick

```bash
git cherry-pick SHA
# on conflict: resolve, git add, git cherry-pick --continue  (or --abort)
```

## Stashing

```bash
git stash push -m "description"
git stash list
git stash pop        # apply + remove
git stash apply      # apply + keep
```

## Tags

```bash
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0
git push origin --delete v1.2.0           # ASK FIRST
```

## Undoing safely

```bash
git reset --soft HEAD~1     # undo last commit, keep staged
git reset HEAD~1            # undo last commit, keep unstaged
git revert SHA              # safe undo of a pushed commit
git push --force-with-lease # only when approved
```

## Inspecting history

```bash
git log --oneline --graph --all -20
git show SHA
git diff main..feature-branch --stat
git log -S "search_string" --oneline     # find commit that touched a string
```

