---
name: commit-push-pr
description: >
  Two-mode inner development loop. Commit mode (default): fast, gate-free
  atomic commits — stage, write a conventional commit message per task,
  push. Ship mode: when the feature is complete and a pull request should
  open, run the quality gates (the repo's own test and verification skills
  plus the code-reviewer, security-auditor, perf-audit, and doc-sync
  agents) over the full branch diff, fix findings, and open a PR with a
  structured description and risk assessment. Uses the commit-message and
  pr-description skills. Use when asked to commit changes, or to ship /
  open a PR.
---

# Commit, Push, PR — Two-Mode Inner Loop

Intermediate task commits should be fast and frequent. Quality gates are
expensive and belong at the pull-request boundary, where they review the
whole feature at once. This skill therefore has two modes:

## Mode Selection

- **Commit mode (default)** — any intermediate commit: tasks remain in the
  feature, or the user asked to commit ("commit this", "commit and push")
  without mentioning a PR.
- **Ship mode** — the feature is complete and a PR should open. Enter ship
  mode when the user asks for a PR ("open a PR", "ship this", "create a
  pull request", or provided a PR title), OR when you judge the feature
  complete on your own — e.g., every task in the active dev-tasks file is
  done and no follow-up work remains.

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
state (like Jira), not code. When a task's context-file update rides along
with its code commit, stage it silently — the commit message must carry
zero trace of the dev-tasks system: no `.claude/dev/` paths, no dev-tasks
task IDs, no bookkeeping mentions ("update dev-tasks", "sync task
context"). The message describes the code change only. (External tracker
IDs — real Jira keys, GitHub issues — are still fine.)

### Step 5: Push

```bash
git push -u origin "$(git rev-parse --abbrev-ref HEAD)"
```

If push fails due to divergence, report the conflict — don't force push.

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
- If `gh` CLI isn't available, provide the manual PR creation URL
