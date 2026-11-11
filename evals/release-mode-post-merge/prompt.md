---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch project whose release PR has just merged, then close it
out:

1. In the current (empty) working directory, run `git init` and create a `.gitignore`
   containing `.claude/dev/`, an initial commit on `main` (the default
   branch), and a `dev` branch (the integration branch).
2. Create one task folder,
   `.claude/dev/fixes/2026-10-06-gh-36-cap-sets/`, with `plan.md`,
   `context.md`, and a `tasks.md` that has `## Release Steps` with
   `- [ ] apply migration 0042_cap_sets to production` and ends in
   `Status: merged to dev (4be9a1c), not released`.
   Write the `.claude/dev/` files with shell commands (e.g. heredocs);
   the sandbox may refuse the Write tool under `.claude/`.
3. Then: **the release PR #50 (dev → main) just merged; it closes #36.
   Close out the release.** (`gh` cannot reach GitHub in this sandbox.)
