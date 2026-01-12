---
name: pr
description: "Create a pull request on GitHub using gh CLI"
allowed-tools: Bash
---

# Pull Request Command

Create a pull request for the current branch.

## Instructions

1. First, gather context by running in parallel:
   - `git status` - check for uncommitted changes
   - `git branch --show-current` - get current branch name
   - `git log --oneline -10` - see recent commits

2. Check if current branch is pushed:
   - Run `git rev-parse --abbrev-ref --symbolic-full-name @{u}` to check upstream
   - If no upstream, push first with `git push -u origin HEAD`

3. Get the diff from the base branch:
   ```bash
   git log origin/master..HEAD --oneline
   git diff origin/master...HEAD --stat
   ```

4. Analyze ALL commits that will be in the PR (not just the latest)

5. Create the PR using gh CLI with HEREDOC for body:

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

6. Return the PR URL to the user

## Important

- If there are uncommitted changes, ask if the user wants to commit first
- The PR title should be concise and descriptive
- The summary should explain the "why" not just the "what"
- Always include a test plan section
- If the branch is already in a PR, inform the user and provide the existing PR URL
