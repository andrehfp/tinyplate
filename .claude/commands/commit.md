---
name: commit
description: "Stage all changes and create a commit with an AI-generated message"
allowed-tools: Bash
---

# Commit Command

Create a well-formatted commit for all staged and unstaged changes.

## Instructions

1. Run `git status` to see all changes (do NOT use -uall flag)
2. Run `git diff` to see unstaged changes
3. Run `git diff --cached` to see staged changes
4. Run `git log --oneline -5` to see recent commit message style
5. Analyze all changes and draft a commit message that:
   - Summarizes the nature of changes (feature, fix, refactor, docs, test, etc.)
   - Uses imperative mood ("Add feature" not "Added feature")
   - Is concise (1-2 sentences) focusing on the "why" not just "what"
   - Follows the repository's commit style from recent commits
6. Stage all changes with `git add -A`
7. Create the commit using a HEREDOC for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
Your commit message here

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

8. Run `git status` to verify the commit succeeded

## Important

- Do NOT commit files that contain secrets (.env, credentials, API keys)
- If pre-commit hooks fail, fix the issues and create a NEW commit (don't amend)
- If there are no changes to commit, inform the user
