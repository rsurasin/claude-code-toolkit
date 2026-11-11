---
name: commit-push-pr
description: >
  Two-mode inner development loop. Commit mode (default): fast, gate-free
  atomic commits — stage, write a conventional commit message per task,
  push. Ship mode: when the feature is complete and a pull request should
  open, run the built-in /simplify first on code changes, then the quality
  gates (the repo's own test and verification skills, the code-reviewer,
  security-auditor, perf-audit, and doc-sync agents, and the built-in
  /code-review and /security-review skills side by side) over the full
  branch diff, triage and merge their findings, fix introduced ones, and
  open a PR with a structured description and risk assessment. Release
  mode, ship mode's variant for a PR from an integration branch (e.g.
  dev) to the default branch: list the features merged since the last
  release, put them and their release steps in the release PR, and after
  it merges confirm the issues closed and close out the dev-tasks
  folders. Uses the commit-message, pr-description, and dev-tasks
  skills. Use when asked to commit changes, to ship / open a PR, or to
  open or close out a release.
---

# Commit, Push, PR — Two-Mode Inner Loop

Intermediate task commits should be fast and frequent. Quality gates are
expensive and belong at the pull-request boundary, where they review the
whole feature at once. This skill therefore has two modes, plus a release
variant of ship mode:

## Mode Selection

- **Commit mode (default)** — any intermediate commit: tasks remain in the
  feature, or the user asked to commit ("commit this", "commit and push")
  without mentioning a PR.
- **Ship mode** — the feature is complete and a PR should open. Enter ship
  mode when the user asks for a PR ("open a PR", "ship this", "create a
  pull request", or provided a PR title), OR when you judge the feature
  complete on your own — e.g., every task in the active dev-tasks file is
  done and no follow-up work remains.
- **Release mode** — ship mode's variant for a release PR: the head is a
  long-lived integration branch (`dev`, `develop`, `staging`) and the base
  is the default branch. Enter it when the user asks to release, to
  promote the integration branch ("promote dev to main", "open the release
  PR"), or to close out a merged release ("the release PR merged"), and
  when a PR is requested while HEAD is the integration branch and the
  base is the default branch.

When in doubt, use commit mode. A missed ship is recoverable (invoke again
and ask to ship); running eight gates on a half-done feature wastes
minutes and tokens.

If the user provided a title or description in their request, use it as
the PR title/summary override in ship mode Step 5.

---

## Commit Mode

Fast path: no gates (no review agents, no built-in `/code-review`,
`/security-review`, or `/simplify`), no PR. Note that shell variables do
not persist between tool calls — derive values inline in the command
that uses them.

### Step 1: Gather context

```bash
git rev-parse --abbrev-ref HEAD                                  # current branch
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null \
  | sed 's|^origin/||' | grep . || echo "main"                  # base branch (no network)
git status --porcelain                                           # staged/unstaged/untracked
git diff; git diff --staged                                      # the actual changes
```

### Step 2: Branch if on the default branch

If the current branch IS the base branch (e.g., `main`), create a feature
branch before committing — never commit feature work directly to the
default branch:

```bash
git checkout -b <type>/gh-<N>-<short-slug>   # work has GitHub issue #N, e.g. feat/gh-31-password-reset
git checkout -b <type>/<short-slug>          # no issue, e.g. feat/oauth-refresh
```

When the work comes from a GitHub issue (the user names it, or the active
dev-tasks folder carries `gh-<N>-`), put its number in the branch name so
`git branch` and the PR list tie the branch back to the issue. This follows
the same idea as GitHub's own `gh issue develop` naming (`<N>-<title>`), but
GitHub does not link branches to issues by name — the PR body's
`Closes #N` still does the linking. Quick chores with no issue keep
`<type>/<short-slug>`. Branch names never carry a date, even when the
dev-tasks folder for the same work does (`2026-10-04-gh-31-password-reset/`
pairs with `feat/gh-31-password-reset`, not
`feat/2026-10-04-gh-31-password-reset`): the folder date records when the
task began; the branch's commits already carry their dates.
`<N>` is digits only, and you write the slug yourself
in `[a-z0-9-]` — never paste issue-title text into the branch name. An issue number is an external tracker ID, so it is
fine in branch names, commits, and PR text; dev-tasks task IDs and
`.claude/dev/` paths are not. Repos that enforce branch names with a hook or
CI pattern should confirm `gh-<N>-` passes.

### Step 3: Group changes into atomic commits

One commit = one task or logical change, so any single commit can be
reverted or rebased without dragging unrelated work along.

- Group the working-tree changes by task. Use the active dev-tasks task
  list to attribute files to tasks when available.
- If everything belongs to the current task: one commit.
- If changes span multiple tasks or features: split into separate commits,
  staged and committed in sequence — one group at a time.
- If a single file mixes unrelated changes and cannot be cleanly
  attributed, ask the user how to split it. Never bundle unrelated changes
  into one commit.

### Step 4: Stage and commit each group

For each group, in sequence:

1. Stage by naming files explicitly (`git add <files>`, never `git add -A`)
2. Never stage: `.env` files, secrets, large binary files, build artifacts
   (`dist/`, `build/`, `target/`, `__pycache__/`, `node_modules/`), or
   generated files that should be in `.gitignore`
3. Write the message using the `commit-message` skill conventions — type,
   scope, imperative subject <72 chars, body with what/why for non-trivial
   changes. Scope the subject to that one group's task. Pass the message
   with `git commit -F - <<'EOF'` … `EOF` rather than `-m "<message>"`, so
   backticks and `$(...)` in it stay literal.

**Dev-tasks separation:** `.claude/dev/` task files are internal tracking
state — gitignored working notes, never staged or committed. If they
appear as untracked/modified, `.claude/dev/` is missing from the
`.gitignore`; add it rather than staging them. The commit message must
carry zero trace of the dev-tasks system: no `.claude/dev/` paths, no
dev-tasks task IDs, no bookkeeping mentions ("update dev-tasks", "sync
task context"). The message describes the code change only. (External
tracker IDs — real Jira keys, GitHub issues — are still fine.)

### Step 5: Push

```bash
git push -u origin "$(git rev-parse --abbrev-ref HEAD)"
```

If push fails due to divergence, report the conflict — don't force push.

**Task status:** if an active dev-tasks folder exists for this work, keep
its `tasks.md` status line current (states and transitions are defined in
the dev-tasks skill's Status and Release Tracking section). Set
`Status: in progress` when the line is missing or the folder's PR was
closed without merging; leave `PR open (#N)` as is when this branch
already has an open PR. If the folder says the work already merged or
released, stop and ask: new commits belong on a new branch with its own
folder (as for release-mode gate fixes, see `release-mode.md`). Never create
a folder just to hold a status. The status update is gitignored working
state and is never staged.

### Step 6: Report

```
## Committed

**Commits:**
- <type>(<scope>): <subject>
- <type>(<scope>): <subject>   (if split)
**Branch:** <branch> (pushed)
**Gates:** not run — intermediate commit; gates run at PR time (ship mode)
```

---

## Ship Mode

In ship mode, read [`ship-mode.md`](ship-mode.md) in this skill's
directory **before taking any ship-mode step**, and follow it. In short:
commit outstanding work, confirm the PR base branch, run `/simplify` on
code changes, run every quality gate over `<base>...HEAD`, triage the
findings into one list, fix introduced findings above the threshold and
contained ones below it, push, and open the PR with a risk assessment
and an Unresolved findings list. The Rules below apply in every mode.

---

## Release Mode

In release mode, read [`release-mode.md`](release-mode.md) in this
skill's directory and follow it. In short: list the PRs merged into the
integration branch since the last release, map each to its dev-tasks
folder, run the gates over the release diff, and open the release PR with
the feature list, release steps, and a `Closes #N` per issue. After it
merges, confirm the issues closed, do (with per-step approval) or hand
off the release steps, and mark the folders `released`.

---

## Rules

- Gates run once, at the PR boundary, over the full branch diff — never
  per intermediate commit
- One commit = one task; never bundle unrelated changes
- Commit messages carry zero trace of the dev-tasks system
- Ship mode runs the review agents and the built-in `/code-review` and
  `/security-review` side by side — never one instead of the other; an
  unavailable built-in skill is `N/A`, which keeps the PR above LOW
- Never run `/code-review ultra` automatically — it is user-triggered
  and billed
- Introduced findings above the threshold (any CRITICAL or HIGH, or 2+
  MEDIUM) are
  fixed before the PR opens — never shipped with a risk label as a
  substitute for fixing; existing findings are reported or filed, not
  blocking (except an existing CRITICAL security finding)
- Below the threshold, contained fixes (needed to deliver what the
  issue and tasks asked for, no design or contract change, too small
  for its own issue)
  are made in this PR too; every introduced finding left unfixed is
  listed under Unresolved findings in the PR with its reason
- Never publish an existing security finding's location, class, or
  exploit path in a PR body, commit message, or public issue — nor an
  unfixed introduced one's
- Never force push
- Never commit `.env`, secrets, or large binary files
- Never commit feature work directly to the default branch
- If there are merge conflicts, stop and report
- On a hard stop (failing tests, existing critical security finding,
  serious perf regression), never push without explicit user approval;
  an introduced CRITICAL is fixed, not shipped
- Ask for the PR base branch unless the user stated it — before the
  gates run (Step 1a)
- Release PRs list features, PR and issue numbers, and release steps,
  never dev-tasks paths, folder names, task IDs, or secret values
- Never commit onto the integration or default branch in release mode
- Never mark a task folder `released` before its release PR merges, and
  never run a release step without the user's explicit approval for that
  step
- If `gh` CLI isn't available, provide the manual PR creation URL
- Pass PR bodies with `--body-file`, never through shell interpolation
