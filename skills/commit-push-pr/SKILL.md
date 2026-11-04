---
name: commit-push-pr
description: >
  Two-mode inner development loop. Commit mode (default): fast, gate-free
  atomic commits — stage, write a conventional commit message per task,
  push. Ship mode: when the feature is complete and a pull request should
  open, optionally run the built-in /simplify first, then run the quality
  gates (the repo's own test and verification skills, the code-reviewer,
  security-auditor, perf-audit, and doc-sync agents, and the built-in
  /code-review and /security-review skills side by side) over the full
  branch diff, triage and merge their findings, fix introduced ones, and
  open a PR with a structured description and risk assessment. Uses the
  commit-message and pr-description skills. Use when asked to commit
  changes, or to ship / open a PR.
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

### Step 1a: Confirm the PR base branch

Every gate and the triage step compare against `<base>`, so confirm it
before anything runs. If the user did not explicitly state which branch
to merge into, ask now. Suggest the repo's default branch (from
`git symbolic-ref refs/remotes/origin/HEAD`) as the recommended option,
and list other long-lived branches if present (`develop`, `release/*`,
`staging`). Never silently assume the default branch.

### Step 1b: Optional `/simplify` pre-pass

Consider running the built-in `/simplify` before the gates, so cleanup
(reuse, simplification, efficiency) lands before review rather than as
review findings. Run it when the branch adds or restructures non-trivial
code; skip it for docs-only or tiny diffs, or when the skill isn't
available. The tree is clean after Step 1, so point it at the feature
explicitly — name the range `<base>...HEAD` and the changed files
(`git diff --name-only <base>...HEAD`) when invoking it.

`/simplify` applies its fixes to the working tree. Review them, then
commit them atomically before Step 2 starts — Commit Mode Steps 3–4,
e.g. `refactor(<scope>): simplify <area>`; they are pushed with the rest
of the branch in Step 4. List the simplify commit in the final report so
the user sees what changed. The gates then review the simplified code.
`/simplify` is a pre-step, not a gate: it gets no row in the risk table.

### Step 2: Run quality gates over the full branch diff

Gates review the **entire feature** — `<base>...HEAD` — not the last
commit. Discover the repo's own verification capabilities, then run all
gates, in parallel where possible — spawn the agents first, then invoke
the built-in skills while the agents run.

Tell every spawned agent that it runs **unattended and report-only**:
it must not ask the user anything (record the question as a gap in its
report instead) and must not edit files — fixes happen in Step 3.

1. **Tests** — If the target repo defines a qa/test skill or agent in its
   `.claude/` directory, run that. Otherwise run the test suite using the
   commands documented in the repo's CLAUDE.md or package scripts. If
   neither exists, record `NOT RUN`.
2. **Runtime verification** — If the repo defines a verify skill or agent
   in its `.claude/` directory, run that. Otherwise record `NOT RUN`.
3. **`code-reviewer` agent** — review the branch diff (pass
   `<base>...HEAD`).
4. **`security-auditor` agent** — security audit of the branch diff, in
   gate mode (it records questions under Scanning Gaps).
5. **`perf-audit` agent** — performance audit of the branch diff.
6. **`doc-sync` agent** — report-only check of whether docs drifted from
   the changes; it proposes edits but applies none.
7. **Built-in `/code-review`** — invoke the skill with an **explicit
   effort level and target**: `high` and the range `<base>...HEAD` (or
   the current feature branch's name, if the skill only takes a branch —
   never the base branch's name, which reviews the base itself). Never
   omit the
   level: with none given, the skill reuses the last level the user
   typed, which may be `ultra`. Never omit the target either: on a
   pushed branch the skill may otherwise diff against the branch's own
   upstream, which is an empty range. Check the range its report says it
   reviewed — if it is empty or not `<base>...HEAD`, record `NOT RUN`,
   never PASS. **Never run `/code-review ultra` automatically** — it
   runs in the cloud and is billed to the user, so only the user
   triggers it. Don't pass `--fix` (fixes go through Step 3) or
   `--comment` (no PR exists yet).
8. **Built-in `/security-review`** — invoke the skill; it reviews what
   the current branch introduces relative to `origin/HEAD`. If it errors
   because `origin/HEAD` is unset, run
   `git remote set-head origin --auto` and retry once. If the confirmed
   `<base>` isn't the remote's default branch, it covers a different
   range: Step 2b re-tags its findings, and the risk justification
   sentence states the mismatch.

Agents and built-in skills run side by side, never one instead of the
other: side-by-side runs showed each catches defects the other misses.
The agents bring installed tooling, existing-code coverage, performance,
docs, test coverage, and `.claude/rules/`; `/code-review` traces callers
and audits removed behavior; `/security-review` confirms every finding
with a strict false-positive check. Step 2b merges them into one verdict.

If a built-in skill isn't available in the running Claude Code version
(not in the available skills list, or the Skill tool can't invoke it),
record it as **`N/A`**. `N/A` is a verification gap and raises the risk
level — a PR with an `N/A` gate is never LOW (see the `pr-description`
floors).

Capture each gate's outcome verbatim enough to normalize later. Do not
skip gates in ship mode; a gate that is available but didn't run (or
errored) is recorded as `NOT RUN`, which raises the risk level.

### Step 2b: Triage — merge every reviewer's findings

Triage the findings from all gates, agents and skills alike, into one
list before evaluating, using the `pr-description` Severity Scale. You
wrote this branch, so guard against grading your own work leniently:
**when unsure, choose `introduced` and the higher severity**, and never
lower a reviewer's severity or re-tag its finding `existing` without
recording the reason — in the PR's Notes for Reviewers, except for a
security finding re-tagged `existing`, whose reason goes in the Step 6
report only (the PR may say "a security finding was re-triaged and
reported privately", with no location or class).

1. **Assign severities.** Map every finding onto the Severity Scale by
   consequence:
   - `/code-review` findings carry no severity: assign one from the
     failure scenario (a reachable race that leaves inconsistent state
     is HIGH, however small the fix).
   - `perf-audit`: Critical → HIGH (CRITICAL for a serious regression),
     Important → MEDIUM, Optimization → LOW.
   - `doc-sync`: CRITICAL and STALE → MEDIUM (docs contradict code),
     MISSING and MINOR → LOW.
   - `/security-review` reports HIGH, MEDIUM, or LOW; keep it, or raise
     it (e.g. an exploitable injection to CRITICAL) — never lower it.
2. **Verify `/code-review` findings.** Unlike the agents' findings,
   `/code-review` candidates haven't been through a verification pass.
   For each one at MEDIUM or above, trace its failure scenario through
   the actual code — or hand it to a fresh verifier subagent with the
   finding and the code, not this session's reasoning. Drop findings
   whose scenario can't be reproduced, and list the drops in the Step 6
   report.
3. **Merge duplicates.** When reviewers report the same defect (same
   root cause, same location), keep one finding with the **highest**
   severity any source gave it and name **every** source, e.g.
   `[HIGH] (code-reviewer, /code-review)`. Exception: a finding that
   only `security-auditor` reported, and capped as `unconfirmed`, stays
   at most MEDIUM.
4. **Tag introduced vs existing.** **Introduced**: the defect lives in
   lines this branch adds or changes, or the branch causes it.
   **Existing**: already present at `git merge-base <base> HEAD` and not
   worsened. Keep the tags of agents that emit them (`code-reviewer`,
   `security-auditor`) after a sanity check; tag the rest yourself
   (`/code-review`, `perf-audit`, `doc-sync`) with
   `git diff <base>...HEAD` and `git blame`. `/security-review` findings
   are introduced when `<base>` is the remote's default branch;
   otherwise tag them like `/code-review` findings and drop any outside
   `<base>...HEAD` from its row (route them per item 6). When sources
   disagree, re-check with `git blame`; introduced wins if any source
   shows the branch causes the defect.
5. **Check open issues.** If `gh` works and the remote is on GitHub,
   fetch the open issues once
   (`gh issue list --state open --limit 500 --json number,title`) and
   match existing findings against them locally; link a match. If `gh`
   is unavailable, or the list hit the limit, say so in the Step 6
   report and never file an issue that may duplicate an unchecked one.
6. **Route.** Introduced findings go into Step 3. Existing findings
   don't enter the fix loop and are never fixed inside this PR
   unprompted:
   - **Existing security findings, at any severity,** are reported to
     the user only (in the Step 6 report) — never written into the PR
     body, a commit message, or a public issue with their location,
     class, or exploit path, since that discloses an unpatched
     vulnerability. Recommend a private channel (e.g. a GitHub security
     advisory). An **existing CRITICAL security finding** is still a
     Step 3 hard stop.
   - **Existing non-security findings** go in the PR's Notes for
     Reviewers with
     the matching issue link, or into a new issue (`gh issue create`)
     when the user asked for it or approves.

Then normalize each gate's verdict from its triaged findings using the
`pr-description` verdict normalization table.

### Step 3: Evaluate — fix findings before the PR opens

Using the normalized verdicts from Step 2b, compute the risk level from
the `pr-description` rubric.

**Hard stops — report and ask the user whether to fix first or proceed:**

- Any test failures
- An existing CRITICAL security finding
- A serious performance regression

If the user chooses to fix, follow fix-loop items 3–6 below. To fix an
existing security finding, recommend a separate private fix (security
advisory or private fork) over this PR; if the user wants it here, keep
the commit message and PR text neutral — no vulnerability class,
location, or exploit path (e.g. `fix(<scope>): harden input handling`).

An **introduced CRITICAL** finding of any kind isn't a question: it
goes straight into the fix loop below, and you ask the user only if it
can't be fixed.

**Fix-before-PR loop — findings do not ship:**

If the triaged **introduced** findings include any CRITICAL or HIGH
finding, or two or more MEDIUM findings:

1. Report the findings to the user
2. Append them to the active dev-tasks context file, if one exists
3. Fix the issues
4. Commit the fixes atomically (Commit Mode Steps 3–4, e.g.
   `fix(<scope>): <subject>`) — the gates review `<base>...HEAD`, so an
   uncommitted fix is invisible to them and missing from the PR
5. Re-run Tests and runtime verification, plus only the affected review
   gates — the sources named on the findings you fixed. Each re-run
   still passes `<base>...HEAD` as its range and reference point (so an
   incomplete fix stays `introduced`, and `/code-review`'s range check
   holds); the fix commits' files and the flagged locations are its
   *focus*, not its range (`security-auditor` skips DAST and
   full-codebase tooling on a re-run unless the finding came from that
   tooling). Then re-triage, carrying forward every earlier finding the
   re-run didn't cover, so each row reflects the whole branch.
6. Repeat until the threshold is no longer met, or the user says to
   proceed anyway. After three fix rounds, stop and ask the user whether
   to continue, listing the remaining findings.

LOW-severity findings, or a single MEDIUM (which makes the PR HIGH
(findings)), are acknowledged in the PR description and proceed; clean
gates proceed directly. `NOT RUN` and `N/A` gaps don't block — they are
reflected honestly in the risk level instead. Existing findings never
block, except an existing CRITICAL security finding (hard stop above);
Step 2b already routed them.

### Step 4: Push the final branch

Push the branch, including any simplify and fix commits from Steps 1b
and 3 (Commit Mode Step 5). The base branch was confirmed in Step 1a;
if the user changed it since, re-run the gates against the new base.
If the repo has no remote, report that the push and PR aren't possible
and hand the user the drafted PR description from Step 5.

### Step 5: Open PR

Using the `pr-description` skill (including its Risk Assessment section):

1. Analyze all commits on this branch vs the confirmed base
2. Generate the PR description with:
   - Summary (1-3 sentences, PM-readable)
   - Risk assessment — level, reversibility, and the evidence table filled
     from the final Step 2/2b/3 gate results
   - Under Notes for Reviewers: existing non-security findings from
     Step 2b with issue links, and any severity or tag a triage changed,
     with the reason (security findings re-tagged `existing` excepted —
     their reasons stay in the Step 6 report, per Step 2b)
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
**Gates:** <n> ran clean, <n> fixed then clean, <n> not run, <n> N/A
**Commits added by ship mode:** <simplify and fix commits, or none>
**Existing issues:** <n> reported, <n> linked or filed
**Reported privately:** <existing security findings (file:line, class, severity) and security re-triage reasons; recommend a security advisory>
**Dropped candidates:** <unreproduced /code-review findings; security-auditor drops (see its report)>
**PR:** <url or instructions>
**Files:** <count> changed, <insertions> insertions, <deletions> deletions
```

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
- Never publish an existing security finding's location, class, or
  exploit path in a PR body, commit message, or public issue
- Never force push
- Never commit `.env`, secrets, or large binary files
- Never commit feature work directly to the default branch
- If there are merge conflicts, stop and report
- On a hard stop (failing tests, existing critical security finding,
  serious perf regression), never push without explicit user approval;
  an introduced CRITICAL is fixed, not shipped
- Ask for the PR base branch unless the user stated it — before the
  gates run (Step 1a)
- If `gh` CLI isn't available, provide the manual PR creation URL
