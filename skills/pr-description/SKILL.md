---
name: pr-description
description: >
  Generate structured pull request descriptions from diffs or branch changes.
  Produces a clear summary, change list, testing notes, and review guidance.
  Use when creating a PR or when asked to write a PR description.
---

# PR Description Skill

Generate clear, structured pull request descriptions that help reviewers
understand what changed, why, and how to verify it.

## Process

1. Read the diff: `git diff main...HEAD` (or the target branch)
2. Read any related CLAUDE.md rules for context
3. Identify the category and scope of changes
4. Generate the description

## Template

```markdown
## Summary

[1-3 sentences: what this PR does and why. Should answer "what problem does
this solve?" without requiring the reviewer to read the diff.]

## Changes

[Grouped by area, not by file. Each item explains the *what* and *why*,
not just the file name.]

### [Area 1, e.g., "Authentication"]
- [Change description with motivation]
- [Change description with motivation]

### [Area 2, e.g., "API Layer"]
- [Change description with motivation]

## Testing

[How to verify this works. Be specific enough that someone unfamiliar with
the feature can test it.]

1. [Step 1]
2. [Step 2]
3. [Expected result]

## Screenshots / Recordings

[If UI changed, note that screenshots should be added. If no UI change,
say "N/A — no UI changes".]

## Notes for Reviewers

[Anything the reviewer should pay special attention to: tricky logic,
intentional deviations from patterns, known limitations, follow-up work.]

## Checklist

- [ ] Tests added/updated for changed logic
- [ ] Documentation updated (if public API changed)
- [ ] No secrets or credentials in the diff
- [ ] Breaking changes noted (if any)
- [ ] Tested locally before opening PR
```

Extend this checklist with project-specific items in your `.github/PULL_REQUEST_TEMPLATE.md` or `.claude/rules/`.

## Rules

- The summary should be understandable by a PM, not just engineers
- Group changes by logical area, not by file
- Testing steps should be copy-pasteable
- If the PR is large (>15 files), add a "How to Review" section suggesting
  an order to read the files
- If the PR introduces a new pattern, call it out explicitly
- Reference task IDs from the roadmap if applicable
- Keep it concise — a PR description longer than the diff is a red flag
