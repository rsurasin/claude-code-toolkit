---
max_turns: 30
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch project with dev-tasks folders in three states, then
resume work:

1. Create a temporary directory with `git init`, a `.gitignore` containing
   `.claude/dev/`, an initial commit on `main`, and a long-lived `dev`
   branch (the integration branch; `main` is the default branch).
2. Create three task folders, each with `plan.md`, `context.md`, and a
   `tasks.md` whose last line is the given status:
   - `.claude/dev/features/2026-10-04-gh-31-password-reset/` ending in
     `Status: PR open (#41)`
   - `.claude/dev/fixes/2026-10-06-gh-36-cap-sets/` ending in
     `Status: merged to dev (4be9a1c), not released`, with a
     `## Release Steps` section listing
     `- [ ] apply migration 0042_cap_sets`
   - `.claude/dev/features/2026-10-20-gh-48-streak-badges/` ending in
     `Status: in progress`
3. Then, in a fresh session: **PR #41 was merged into dev as commit
   9f3c2e1. Catch me up on where things stand.**
