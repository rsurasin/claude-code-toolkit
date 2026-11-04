---
name: commit-push-pr
description: >
  Two-mode inner development loop. Commit mode (default): fast, gate-free
  atomic commits — stage, write a conventional commit message per task,
  push. Ship mode: when the feature is complete and a pull request should
  open, run the quality gates (the repo's own test and verification skills
  plus the code-reviewer, security-auditor, perf-audit, and doc-sync
  agents) over the full branch diff, fix findings, and open a PR with a
  structured description and risk assessment. Release mode, ship mode's
  variant for a PR from an integration branch (e.g. dev) to the default
  branch: list the features merged since the last release, put them and
  their release steps in the release PR, and after it merges confirm the
  issues closed and close out the dev-tasks folders. Uses the
  commit-message, pr-description, and dev-tasks skills. Use when asked to
  commit changes, to ship / open a PR, or to open or close out a release.
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
  PR"), or to close out a merged release ("the release PR merged").

When in doubt, use commit mode. A missed ship is recoverable (invoke again
and ask to ship); running six gates on a half-done feature wastes minutes
and tokens.

If the user provided a title or description in their request, use it as
the PR title/summary override in ship mode Step 5.

---

## Commit Mode

Fast path: no gates, no PR. Note that shell variables do not persist
between tool calls — derive values inline in the command that uses them.

### Step 1: Gather context

```bash
git rev-parse --abbrev-ref HEAD                                  # current branch
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null \
  | sed 's|^origin/||' || echo "main"                            # base branch (no network)
git status --porcelain                                           # staged/unstaged/untracked
git diff; git diff --staged                                      # the actual changes
```

### Step 2: Branch if on the default branch

If the current branch IS the base branch (e.g., `main`), create a feature
branch before committing — never commit feature work directly to the
default branch:

```bash
git checkout -b <type>/<short-slug>   # e.g., feat/oauth-refresh
```

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
   changes. Scope the subject to that one group's task.

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
released, stop and ask: new commits belong on a new branch. Never create
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

The feature is complete. PRs are the last line of defense before code
reaches production: take the time to run the gates rather than optimizing
for speed.

### Step 1: Commit outstanding work

If uncommitted changes exist, run Commit Mode Steps 1–5 first (atomic
grouping included). Ship mode starts from a clean tree.

### Step 2: Run quality gates over the full branch diff

Gates review the **entire feature** — `<base>...HEAD` — not the last
commit. Discover the repo's own verification capabilities, then run all
gates, in parallel where possible:

1. **Tests** — If the target repo defines a qa/test skill or agent in its
   `.claude/` directory, run that. Otherwise run the test suite using the
   commands documented in the repo's CLAUDE.md or package scripts. If
   neither exists, record `NOT RUN`.
2. **Runtime verification** — If the repo defines a verify skill or agent
   in its `.claude/` directory, run that. Otherwise record `NOT RUN`.
3. **`code-reviewer` agent** — review the branch diff.
4. **`security-auditor` agent** — security audit of the branch diff.
5. **`perf-audit` agent** — performance audit of the branch diff.
6. **`doc-sync` agent** — check whether docs drifted from the changes.

The review and audit gates are dedicated agents on purpose — do not
replace them with in-context skill calls (including native code-review /
security-review): a skill runs in the context that just wrote the code
and inherits its assumptions, while a spawned agent reviews the diff cold
and returns a verdict the rubric can normalize.

Capture each gate's outcome verbatim enough to normalize later. Do not
skip gates in ship mode; a gate that didn't run is recorded as `NOT RUN`,
which raises the risk level.

### Step 3: Evaluate — fix findings before the PR opens

Normalize every gate result using the verdict normalization table in the
`pr-description` skill, then compute the risk level from its rubric.

**Hard stops — report and ask the user whether to fix first or proceed:**

- Any test failures
- A critical security vulnerability
- A serious performance regression

**Fix-before-PR loop — findings do not ship:**

If the gates produced any HIGH-severity finding, or two or more
MEDIUM-severity findings:

1. Report the findings to the user
2. Append them to the active dev-tasks context file, if one exists
3. Fix the issues
4. Re-run the affected gates (only the ones whose findings you fixed)
5. Repeat until the threshold is no longer met, or the user says to
   proceed anyway

Only LOW-severity findings (acknowledged in the PR description) or clean
gates proceed directly. `NOT RUN` gaps don't block — they are reflected
honestly in the risk level instead.

### Step 4: Confirm the PR base branch

If the user did not explicitly state which branch to merge into, ask
before creating the PR. Suggest the repo's default branch (from
`git symbolic-ref refs/remotes/origin/HEAD`) as the recommended option,
and list other long-lived branches if present (`develop`, `release/*`,
`staging`). Never silently assume the default branch.

### Step 5: Open PR

Using the `pr-description` skill (including its Risk Assessment section):

1. Analyze all commits on this branch vs the confirmed base
2. Generate the PR description with:
   - Summary (1-3 sentences, PM-readable)
   - Risk assessment — level, reversibility, and the evidence table filled
     from the final Step 2/3 gate results
   - Changes grouped by area
   - Testing steps
   - Notes for reviewers
   - Checklist

If the user provided a title/description override, use it instead of
generating one.

```bash
gh pr create --title "<title>" --body "<description>" --base <confirmed-base>

# Or if gh isn't available, provide the URL
echo "Create PR at: https://github.com/$(git remote get-url origin | sed 's/.*github.com[:/]//' | sed 's/.git$//')/compare/<confirmed-base>...<branch>"
```

If the branch already has an open PR, push without creating a new one and
update the PR description's risk assessment with the latest gate results.

Once the PR exists, set the active dev-tasks folder's status line to
`Status: PR open (#N)` and append the transition to its `context.md`
Session Log. Before opening the PR, make sure the folder's
`## Release Steps` lists anything the release will need (deploy pairings,
migrations, flags), or `None`.

### Step 6: Report

```
## Shipped

**Branch:** <branch> → <base>
**Risk:** <LOW / MEDIUM / HIGH / CRITICAL> (<one-line justification>)
**Gates:** <n> ran clean, <n> fixed then clean, <n> not run
**PR:** <url or instructions>
**Files:** <count> changed, <insertions> insertions, <deletions> deletions
```

---

## Release Mode

A release PR promotes an integration branch (`<integration>`, e.g. `dev`)
to the default branch (`<default>`, e.g. `main`). The feature PRs merged
into the integration branch have not shipped yet: GitHub fires their
`Closes #N` keywords only when the change reaches the default branch, and
their release-time steps are still waiting. Release mode is ship mode
plus a feature list and a closeout, in two phases: opening the release
PR, and closing out after it merges.

### Opening the release PR

1. **Confirm the branches.** Ask for the integration and default branches
   unless the user stated them, suggesting the repo's default branch
   (`git symbolic-ref refs/remotes/origin/HEAD`) as the base. The working
   tree must be clean; never commit feature work straight onto the
   integration branch. If uncommitted changes exist, stop and ask.
2. **List the PRs merged since the last release.**

   ```bash
   git fetch origin
   # last release: the most recent merged PR from integration into default
   gh pr list --base <default> --head <integration> --state merged \
     --limit 20 --json number,mergedAt
   # feature PRs merged into integration after it (latest mergedAt above)
   gh pr list --base <integration> --state merged --limit 200 \
     --search "merged:>LAST_RELEASE_MERGED_AT" \
     --json number,title,headRefName,mergedAt,mergeCommit,body
   # cross-check against what the release actually carries
   git log --oneline origin/<default>..origin/<integration>
   ```

   With no earlier release PR, everything in
   `git log origin/<default>..origin/<integration>` is in scope. A PR that
   one source shows and the other doesn't is listed and flagged, never
   dropped. For each PR, collect the issues it closes from the
   `Closes` / `Fixes` / `Resolves #N` keywords in its body and commits
   (GitHub does not link them for a non-default base, so read the text).
3. **Map PRs to task folders and gather release steps**, using the
   dev-tasks skill's Release closeout (steps 1–2). PRs with no folder are
   listed with "release steps unknown" and the user is asked.
4. **Run the gates** — Ship Mode Steps 2–3 over
   `origin/<default>...origin/<integration>`. Each feature was gated on its
   own, but not together with the others. A migration or other one-way
   step among the release steps makes the release a one-way door in the
   risk assessment.
5. **Open the release PR** with the `pr-description` skill, adding these
   sections after the Summary:

   ```markdown
   ## Features in This Release
   - #41 Password reset by email (closes #31)
   - #43 Cap sets_completed at the prescribed count (closes #36)
   - #45 Retry failed webhook deliveries (release steps unknown; confirm with the author)

   ## Release Steps
   - [ ] #41: deploy together with org/frontend#56 (the reset screen calls the new endpoint)
   - [ ] #43: apply migration 0042_cap_sets before the new build takes traffic
   - #45: none recorded

   Closes #31
   Closes #36
   ```

   Write features and steps in product terms. The release PR carries
   features, PR and issue numbers, and release steps only: never
   `.claude/dev/` paths, folder names, dev-tasks task IDs, or bookkeeping
   mentions. One `Closes #N` line per issue, so the issues close when the
   release merges. Open it with
   `gh pr create --base <default> --head <integration>`, then report as in
   Ship Mode Step 6, adding the feature count.

Task folders keep their `merged to <integration> ..., not released`
status until the release PR merges.

### After the release PR merges

Run this when the user says the release merged, or when a session finds
it merged (`gh pr view <N> --json state,mergedAt,mergeCommit`). The same
steps apply to a feature PR merged straight into the default branch.

1. **Confirm the merge.** If the PR is not merged, stop. If `gh` is
   unavailable, the user's report of the merge stands; say it was not
   checked.
2. **Confirm every issue closed.** Run `gh issue view <N> --json state`
   for each `Closes #N`. Report any still open, and close it with
   `gh issue close <N> --comment "Released in #<release PR>"` only once
   the user agrees. If `gh` is unavailable, report the issues as
   unconfirmed; never claim they closed.
3. **Do or hand off each release step.** Deploys, migrations, and flag
   flips change shared environments and are often irreversible: run one
   only with the user's explicit approval in this conversation, otherwise
   hand it off as a checklist naming who does it. Never report a step
   done without checking it.
4. **Close out the folders**, using the dev-tasks skill's Release
   closeout (step 3): each becomes `released (#<release PR>, <date>)`.
5. **Report:**

   ```
   ## Released

   **Release PR:** #<N> (<integration> → <default>), merged <date>
   **Features:** <count> (#41, #43, ...)
   **Issues:** <n> confirmed closed, <n> still open or unconfirmed (#...)
   **Release steps:** <n> done, <n> handed off (to whom), <n> pending
   **Task folders:** <n> marked released, <n> PRs with no folder
   ```

---

## Rules

- Gates run once, at the PR boundary, over the full branch diff — never
  per intermediate commit
- One commit = one task; never bundle unrelated changes
- Commit messages carry zero trace of the dev-tasks system
- Findings above the threshold (any HIGH, or 2+ MEDIUM) are fixed before
  the PR opens — never shipped with a risk label as a substitute for
  fixing
- Never force push
- Never commit `.env`, secrets, or large binary files
- Never commit feature work directly to the default branch
- If there are merge conflicts, stop and report
- On a hard stop (failing tests, critical security finding, serious perf
  regression), never push without explicit user approval
- Ask for the PR base branch unless the user stated it
- Release PRs list features, PR and issue numbers, and release steps,
  never dev-tasks paths, folder names, or task IDs
- Never mark a task folder `released` before its release PR merges, and
  never run a release step that touches a shared environment without
  explicit user approval
- If `gh` CLI isn't available, provide the manual PR creation URL
