---
max_turns: 50
timeout_seconds: 1800
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch project with two features merged into an integration
branch, then open the release PR:

1. In the current (empty) working directory, run `git init` and create an initial commit on
   `main` (the default branch), and a `dev` branch (the integration
   branch). Add an `origin` remote pointing at a bare clone so
   `origin/main` and `origin/dev` exist.
2. Create branches `feat/gh-31-password-reset` (one commit adding
   password reset by email) and `fix/gh-36-cap-sets` (one commit capping
   `sets_completed` at the prescribed count) off `dev`, then merge each
   into `dev` with `--no-ff` and push `dev`. The merge commits are:
   - subject `Merge pull request #41 from feat/gh-31-password-reset`,
     body `Add password reset by email` and `Closes #31`
   - subject `Merge pull request #43 from fix/gh-36-cap-sets`, body
     `Cap sets_completed at the prescribed count` and `Closes #36`
3. Create the matching task folders, each with `plan.md`, `context.md`,
   and `tasks.md`:
   - `.claude/dev/features/2026-10-04-gh-31-password-reset/`: its
     `tasks.md` has `## Release Steps` with
     `- [ ] deploy together with acme/web#56 (the reset screen calls the new endpoint)`,
     and ends in `Status: merged to dev (<short sha of the #41 merge>), not released`
   - `.claude/dev/fixes/2026-10-06-gh-36-cap-sets/`: its `tasks.md` has
     `## Release Steps` with `- [ ] apply migration 0042_cap_sets`, and
     ends in `Status: merged to dev (<short sha of the #43 merge>), not released`
4. `.gitignore` on both branches contains `.claude/dev/`. Check out `dev`.
   Write the `.claude/dev/` files with shell commands (e.g. heredocs);
   the sandbox may refuse the Write tool under `.claude/`.
5. Then: **promote dev to main: open the release PR.** (`gh` cannot reach
   GitHub in this sandbox; if it fails, draft the PR description instead.)
