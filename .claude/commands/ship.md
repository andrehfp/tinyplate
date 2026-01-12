---
name: ship
description: "Commit, push, and create a PR in one command"
allowed-tools: Bash
---

# Ship Command

Commit all changes, push to remote, and create a pull request in one go.

## Instructions

### Step 1: Gather Context

Run these commands in parallel:
- `git status` - check current state (do NOT use -uall flag)
- `git diff` - see unstaged changes
- `git diff --cached` - see staged changes
- `git branch --show-current` - get current branch name
- `git log --oneline -5` - see recent commit style

### Step 2: Commit Changes

1. If there are changes to commit:
   - Analyze all changes and draft a commit message
   - Use imperative mood, be concise, focus on "why"
   - Stage all changes with `git add -A`
   - Create the commit:

```bash
git commit -m "$(cat <<'EOF'
Your commit message here

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

2. If no changes exist, skip to Step 3

### Step 3: Push to Remote

1. Check if branch tracks a remote with `git branch -vv`
2. Push the branch:

```bash
# If no upstream
git push -u origin HEAD

# If already tracking
git push
```

### Step 4: Create Pull Request

1. Get the full diff from base branch:
```bash
git log origin/master..HEAD --oneline
git diff origin/master...HEAD --stat
```

2. Analyze ALL commits that will be in the PR

3. Create the PR:

```bash
gh pr create --title "Your PR title" --body "$(cat <<'EOF'
## Summary
- Brief description of changes
- What problem this solves

## Test plan
- [ ] How to test these changes
- [ ] What to verify

🤖 Generated with [Claude Code](https://claude.ai/code)
EOF
)"
```

4. Return the PR URL to the user

## Important

- Do NOT commit files containing secrets (.env, credentials, API keys)
- If pre-commit hooks fail, fix issues and create a NEW commit
- Never use `git push --force`
- If a PR already exists for the branch, inform the user and provide the URL
- If on master/main branch, warn the user and ask if they want to create a feature branch first
