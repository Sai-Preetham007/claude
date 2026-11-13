---
name: merge-pr
description: >-
  Commits and pushes changes to a new fix/<feature> or feat/<feature> branch,
  raises a pull request to main, merges the PR, and deletes the branch locally
  and remotely. Use when the user runs /merge-pr or asks to commit, raise a PR,
  merge to main, and clean up the branch.
disable-model-invocation: true
---

Commit and push on a new `fix/<feature>` or `feat/<feature>` branch, raise a PR to `main`, merge it immediately, then delete the branch locally and remotely.

## Rules

- Branch name **must** start with `fix/` or `feat/` (e.g. `fix/hero-spacing`); use kebab-case after the slash.
- Base branch is always `main`.
- Repo: `Sai-Preetham007/claude`.
- Required `gh` user for this repo: **`Sai-Preetham007`** (personal). Never raise/merge while another account is active.
- Prefer non-interactive `gh auth switch --user Sai-Preetham007` over `gh auth login` when both accounts are already logged in.
- If switch fails or the account is missing, stop and ask the user to run `gh auth login` (and approve Touch ID/password if the keychain prompts).
- After any keychain / fingerprint prompt, wait for the user to approve, then continue (do not invent tokens or skip auth).
<!-- - Never stage or commit `.gitignore` (leave it unstaged even if modified). -->
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
3. Check `git status -s` and `git diff` (and recent `git log`) to infer a concise `fix/<feature>` or `feat/<feature>` branch name and commit message. Exclude `.gitignore` from the staged set.
4. Execute the full cycle:
   ```bash
   git checkout -b fix/<feature> && \
   git add <relevant-files> && \
   git commit -m "$(cat <<'EOF'
   <commit-message>

   EOF
   )" && \
   git push -u origin fix/<feature> && \
   gh pr create -R Sai-Preetham007/Portfolio --base main --head fix/<feature> --title "<commit-message>" --body "$(cat <<'EOF'
   ## Summary
   <short-summary>

   EOF
   )" && \
   gh pr merge -R Sai-Preetham007/Portfolio --merge --delete-branch && \
   git checkout main && \
   git pull origin main && \
   (git branch -d fix/<feature> 2>/dev/null || true) && \
   git fetch --prune origin
   ```