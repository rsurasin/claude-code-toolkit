---
description: >
  Complete the inner development loop in one command: stage changes, write a
  conventional commit message, push, and open a pull request with a structured
  description. Uses the commit-message and pr-description skills. Inspired by
  Boris Cherny's workflow — he runs this dozens of times daily.
argument-hint: "<optional: PR title or description override>"
---

# Commit, Push, PR — Full Inner Loop

Execute the complete commit-to-PR workflow in one shot. Pre-compute context
to minimize back-and-forth.

## Pre-Compute Context

Gather everything needed before making decisions:

```bash
# Branch info
BRANCH=$(git rev-parse --abbrev-ref HEAD)
BASE_BRANCH=$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}' 2>/dev/null || echo "main")

# What's changed
STAGED=$(git diff --staged --stat)
UNSTAGED=$(git diff --stat)
UNTRACKED=$(git ls-files --others --exclude-standard)

# Diff for commit message generation
FULL_DIFF=$(git diff --staged)
if [ -z "$FULL_DIFF" ]; then
  FULL_DIFF=$(git diff)
fi

# Recent commits on this branch for PR context
RECENT_COMMITS=$(git log --oneline $BASE_BRANCH..HEAD 2>/dev/null | head -20)
```

## Step 1: Stage

If there are unstaged or untracked changes, stage them:

- Review untracked and modified files before staging
- Never stage: `.env` files, secrets, large binary files, build artifacts
  (`dist/`, `build/`, `target/`, `__pycache__/`, `node_modules/`), or
  generated files that should be in `.gitignore`
- If ALL remaining changes are related to the same task, stage them with
  `git add` specifying files by name (not `git add -A`)
- If there's a mix of unrelated changes, ask the user what to include

## Step 2: Commit

Using the `commit-message` skill conventions:

1. Analyze the staged diff
2. Determine the commit type (`feat`, `fix`, `refactor`, etc.)
3. Determine the scope from the primary area changed
4. Write the subject line (imperative mood, <72 chars, lowercase after colon)
5. Write the body if the change is non-trivial (what and why, not how)
6. If the project uses task IDs, include them

Execute:
```bash
git commit -m "<type>(<scope>): <subject>

<body>"
```

## Step 3: Push

```bash
git push -u origin $BRANCH 2>&1
```

If push fails due to divergence, report the conflict — don't force push.

## Step 4: Open PR

Using the `pr-description` skill conventions:

1. Analyze all commits on this branch vs the base
2. Generate the PR description with:
   - Summary (1-3 sentences, PM-readable)
   - Changes grouped by area
   - Testing steps
   - Notes for reviewers
   - Checklist

If the user provided an argument (title/description override), use it as
the PR summary instead of generating one.

Execute:
```bash
# GitHub CLI
gh pr create --title "<title>" --body "<description>" --base $BASE_BRANCH

# Or if gh isn't available, provide the URL
echo "Create PR at: https://github.com/$(git remote get-url origin | sed 's/.*github.com[:/]//' | sed 's/.git$//')/compare/$BASE_BRANCH...$BRANCH"
```

## Step 5: Report

```
## Done

**Commit:** <type>(<scope>): <subject>
**Branch:** <branch> → <base>
**PR:** <url or instructions>
**Files:** <count> changed, <insertions> insertions, <deletions> deletions
```

## Rules

- Never force push
- Never commit `.env`, secrets, or large binary files
- If there are merge conflicts, stop and report
- If `gh` CLI isn't available, provide the manual PR creation URL
- If the branch already has an open PR, push without creating a new one
- Keep the full workflow under 30 seconds for a typical commit
