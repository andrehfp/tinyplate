---
name: push
description: "Push current branch to remote origin"
allowed-tools: Bash
---

# Push Command

Push the current branch to the remote repository.

## Instructions

1. Run `git status` to check:
   - Current branch name
   - If there are uncommitted changes (warn user if so)
   - If branch is ahead of remote

2. Run `git branch -vv` to check if branch tracks a remote

3. If the branch has no upstream:
   ```bash
   git push -u origin HEAD
   ```

4. If the branch already tracks a remote:
   ```bash
   git push
   ```

5. Verify push succeeded and report the result

## Important

- Never use `--force` unless explicitly requested by the user
- If push fails due to conflicts, inform the user and suggest pulling first
- If there are uncommitted changes, ask if the user wants to commit first
