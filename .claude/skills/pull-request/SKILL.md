---
name: pull-request
description: >-
  Pulls the latest changes into the current branch without losing local work: stashes uncommitted changes
  (including untracked files), runs git pull, then pops the stash back. Use when the user runs /pull-request
  or asks to take/pull the latest changes while keeping their local edits.
argument-hint: "[remote branch, optional — defaults to the current branch's upstream]"
disable-model-invocation: true
allowed-tools: Bash(git status *) Bash(git branch --show-current) Bash(git stash *) Bash(git pull *) Bash(git fetch *) Bash(git log *) Bash(git diff *) Bash(git rev-parse *)
---

Pull the latest changes into the current branch while keeping the user's uncommitted work.

Optional target from the user (may be empty): $ARGUMENTS

## Steps

1. **Check state.** Run `git branch --show-current` and `git status --porcelain`.
   - If a merge, rebase or cherry-pick is already in progress, stop and tell the user. Do not stash or pull.

2. **Stash local changes (only if there are any).** If `git status --porcelain` printed anything:
   ```
   git stash push --include-untracked -m "pull-request-skill: <branch> <timestamp>"
   ```
   Then confirm the stash exists with `git stash list | head -1` and note its message. If there were no
   changes, skip the stash and remember that nothing needs to be popped.

3. **Pull.**
   - If `$ARGUMENTS` is empty: `git pull`
   - If `$ARGUMENTS` names a branch: `git pull origin <branch>`

   If the pull fails or leaves merge conflicts, stop. Tell the user which files conflict and that their
   work is still safe in the stash (give the stash message). Do **not** pop the stash on top of a
   conflicted merge, and do not run `git merge --abort`, `git reset` or any destructive command yourself.

4. **Pop the stash back** (only if step 2 created one):
   ```
   git stash pop
   ```
   Before popping, check that the top entry of `git stash list` is the one created in step 2 (match the
   message). If it isn't, use `git stash pop stash@{n}` for the right entry.
   - If the pop reports conflicts, git keeps the stash entry. Tell the user the conflicted files, that the
     stash was **not** dropped, and that they can run `git stash drop` after resolving. Do not drop it yourself.

5. **Report.** Show `git log --oneline -5` and `git status --short`, and summarise in a few lines:
   - how many new commits came in,
   - whether local changes were stashed and restored cleanly,
   - any conflicts the user needs to resolve.

## Rules

- Never use `git stash drop`, `git stash clear`, `git reset --hard`, `git checkout -- .` or `git clean`.
  The user's local changes must never be lost.
- Never commit or push as part of this skill.
