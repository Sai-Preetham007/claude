---
name: raise-pr
description: Commit and push on a new fix/ or feat/ branch, then open a PR to main and leave it open for review (no merge, no branch deletion).
---

Commit and push on a new `fix/<feature>` or `feat/<feature>` branch, raise a PR to `main`, and leave it open for review. Do not merge it and do not delete the branch.

## Rules

- Branch name **must** start with `fix/` or `feat/` (e.g. `fix/hero-spacing`); use kebab-case after the slash.
- Base branch is always `main`.
- Repo: `Sai-Preetham007/claude`.
- Required `gh` user for this repo: **`Sai-Preetham007`** (personal). Never raise a PR while another account is active.
- Prefer non-interactive `gh auth switch --user Sai-Preetham007` over `gh auth login` when both accounts are already logged in.
- If switch fails or the account is missing, stop and ask the user to run `gh auth login` (and approve Touch ID/password if the keychain prompts).
- After any keychain / fingerprint prompt, wait for the user to approve, then continue (do not invent tokens or skip auth).
- Never run `gh pr merge` in this skill, and never delete the branch.
- If the current branch already has an open PR (`gh pr view --json url`), push new commits to it and report the existing URL instead of creating a duplicate.
- Never stage or commit `.gitignore` (leave it unstaged even if modified).
- Never commit secrets (`.env`, credentials, tokens).
- Never force-push; never skip hooks unless the user explicitly asks.
- Follow the user's git/PR rules: status/diff/log before commit; HEREDOC for commit message and PR body.

## Workflow

1. Verify the active GitHub CLI user is the personal account; switch if needed:
   ```bash
   ACTIVE=$(gh api user --jq .login)
   if [ "$ACTIVE" != "Sai-Preetham007" ]; then
     gh auth switch --user Sai-Preetham007
     ACTIVE=$(gh api user --jq .login)
   fi
   ```
   Abort if still not `Sai-Preetham007` and tell the user to run `gh auth login`.
2. Ensure an up-to-date `main`:
   ```bash
   git checkout main && git pull origin main
   ```
   If the GitHub repo is empty (no `main` branch yet), push an initial commit to `main` first, then continue.
3. Check `git status -s` and `git diff` (and recent `git log`) to infer a concise `fix/<feature>` or `feat/<feature>` branch name and commit message. Exclude `.gitignore` from the staged set.
4. Create the branch, commit, and push:
   ```bash
   git checkout -b fix/<feature> && \
   git add <relevant-files> && \
   git commit -m "$(cat <<'EOF'
   <commit-message>

   Co-Authored-By: Claude Haiku 5.5 <noreply@anthropic.com>
   EOF
   )" && \
   git push -u origin fix/<feature>
   ```
5. Open the PR to `main` and stop there:
   ```bash
   gh pr create -R Sai-Preetham007/claude --base main --head fix/<feature> --title "<commit-message>" --body "$(cat <<'EOF'
   ## Summary
   <short-summary>

   ## Test plan
   <how-to-verify>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```
6. Stay on the feature branch and print the PR URL returned by `gh pr create` so the user can review it.
