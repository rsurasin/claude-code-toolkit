Set up a scratch project with two features merged into an integration
branch, then open the release PR:

1. Create a temporary directory with `git init`, an initial commit on
   `main` (the default branch), and a `dev` branch (the integration
   branch). Add an `origin` remote pointing at a bare clone so
   `origin/main` and `origin/dev` exist.
2. On `dev`, add two squash-merge commits and push them:
   - `feat(auth): add password reset by email (#41)` with body
     `Closes #31`, from branch `feat/gh-31-password-reset`
   - `fix(workouts): cap sets_completed at the prescribed count (#43)`
     with body `Closes #36`, from branch `fix/gh-36-cap-sets`
3. Create the matching task folders, each with `plan.md`, `context.md`,
   and `tasks.md`:
   - `.claude/dev/features/2026-10-04-gh-31-password-reset/`: its
     `tasks.md` has `## Release Steps` with
     `- [ ] deploy together with acme/web#56 (the reset screen calls the new endpoint)`,
     and ends in `Status: merged to dev (<sha of the #41 commit>), not released`
   - `.claude/dev/fixes/2026-10-06-gh-36-cap-sets/`: its `tasks.md` has
     `## Release Steps` with `- [ ] apply migration 0042_cap_sets`, and
     ends in `Status: merged to dev (<sha of the #43 commit>), not released`
4. `.gitignore` on both branches contains `.claude/dev/`. Check out `dev`.
5. Then: **promote dev to main: open the release PR.** (`gh` cannot reach
   GitHub in this sandbox; if it fails, draft the PR description instead.)
