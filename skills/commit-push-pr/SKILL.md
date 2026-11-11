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
`git symbolic-ref refs/remotes/origin/HEAD`) as the recommended option;
with no remote, recommend the local `main` (or `master`). List other
long-lived branches if present (`develop`, `release/*`, `staging`) as
alternatives. Never silently assume the default branch.

### Step 1b: `/simplify` pre-pass

Run the built-in `/simplify` before the gates, so cleanup (reuse,
simplification, efficiency) lands before review rather than as review
findings. Whether it runs is a rule, not a judgment call:

- **Run it** whenever `git diff --name-only <base>...HEAD` lists any code
  file: source or test code in any language, scripts, or build and CI
  logic. Size doesn't matter — a one-line code change still runs it.
- **Skip it** only when the diff changes no code: docs, markdown, images,
  or other non-code files. Record `Skipped — <reason>`, e.g.
  `Skipped — docs-only diff`.
- **`N/A`** when the running Claude Code version has no `/simplify`.
  Record `N/A — /simplify not available`.

Keep the outcome for Step 5: the PR description's required **Simplify
pre-pass** line reports it. The tree is clean after Step 1, so point it
at the feature explicitly — name the range `<base>...HEAD` and the changed files
(`git diff --name-only <base>...HEAD`) when invoking it.

`/simplify` applies its fixes to the working tree. Review them, then
commit them atomically before Step 2 starts — Commit Mode Steps 3–4,
e.g. `refactor(<scope>): simplify <area>`; they are pushed with the rest
of the branch in Step 4. List the simplify commit in the final report so
the user sees what changed. The gates then review the simplified code.
If it ran and suggested nothing, the outcome is `Ran — no changes`.
`/simplify` is a pre-step, not a gate: it gets no row in the risk table,
and its outcome never changes the risk level.

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
   Besides the verdict, keep the output it produced: the requests or
   commands it ran, their exit codes or status lines, and the response
   or output bodies. Step 5 (Open PR) passes this to `pr-description`
   for its Screenshots / Output section.
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
the `pr-description` rubric — apply its floors exactly (e.g. Tests
`NOT RUN`, or Security (`security-auditor`) `NOT RUN`, ⇒ at best
**HIGH (unverified)**), never an estimate like "probably MEDIUM".

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
3. Fix the issues — plus, in the same round, every contained
   below-threshold finding (see below)
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

**Below-threshold findings — fix the contained ones too:**

Every other introduced finding (any number of LOW, or a single MEDIUM)
is fixed in this PR when the fix is **contained** — all of:

- **Scope** — it is **within the PR's scope**: the fix is needed for the
  branch to deliver what its work asked for. Measure that against the
  GitHub issue the branch resolves (from `gh-<N>` in the branch name or
  a `Closes #N`, read with `gh issue view <N>`), the active dev-tasks
  `plan.md` and `tasks.md` if present, and the user's request. With no
  issue or task plan, use the branch's commits and the user's request.
  A defect in the requested behavior is in scope even when its fix
  touches code the branch didn't change. Behavior the issue and tasks
  never captured — a new requirement, flow, or user-facing step — is
  out of scope, however small. Example: for a password-reset issue, a
  finding that passwords containing `!` are rejected (even by a shared
  validator the branch never touched) is in scope and fixed here; a
  finding that reset should require an MFA step, which neither the
  issue nor the tasks mention, is out of scope and becomes a follow-up
  issue.
- **No design or contract change** — no architectural or
  design-pattern change, no new dependency, and no change to a public
  API, schema, or config contract.
- **Size** — small enough that it wouldn't warrant its own issue. Size
  measures effort only; it never makes an out-of-scope fix contained.

When unsure whether a finding is contained, treat it as contained — you
wrote this branch, and "not contained" is the answer that saves you
work. If a contained fix turns out bigger once started (it needs a
design change, or spreads well beyond the requested behavior), revert
that fix and record what you found as its reason.

Fold contained fixes into the fix loop's current round, so they cost no
extra gate re-run. If the threshold was never met, run one round of the
loop (items 3–5) for the contained fixes alone. A finding that fails any
of the three tests is **not contained**: leave it unfixed, record which
test it failed and why — citing the issue for a scope call, never a
dev-tasks file (e.g. "out of scope: #31 covers reset by email link,
not MFA", or "needs a schema change") — and offer to file it as an
issue, with that reason as the draft (`gh issue create` once the user approves). Every introduced
finding still unfixed when the PR opens — not contained, the user said
to proceed, or the loop stopped after three rounds — goes in the PR's
**Unresolved findings** list (Step 5) with its reason, and the PR
proceeds. Clean gates proceed directly. `NOT RUN` and `N/A` gaps
don't block — they are reflected honestly in the risk level instead. Existing findings never
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
   - Summary (PM-readable) — the problem and current behavior, then one
     sentence on the fix
   - Risk assessment — level, reversibility, and the evidence table filled
     from the final Step 2/2b/3 gate results, plus the Simplify pre-pass
     line from Step 1b (`Ran — <commit subjects>`, `Ran — no changes`,
     `Skipped — <reason>`, or `N/A — /simplify not available`)
   - Under Notes for Reviewers: the **Unresolved findings** list — every
     introduced finding left unfixed, with severity, every source,
     `file:line`, and the reason (`not contained: <failed test>`, with
     the issue link once filed; `author chose to proceed`; or `stopped
     after 3 fix rounds`), or "None". An unfixed introduced security
     finding is listed neutrally — severity, sources, and "security
     finding, details shared privately" — with its location, class, and
     exploit path in the Step 6 report only
   - Also under Notes for Reviewers: existing non-security findings from
     Step 2b with issue links, and any severity or tag a triage changed,
     with the reason (security findings re-tagged `existing` excepted —
     their reasons stay in the Step 6 report, per Step 2b)
   - Changes — the new behavior first, then the detail grouped by area
     ("No behavior change." in both places for a docs-only or tests-only
     change; "No intended behavior change" plus how it was verified, or
     "(unverified)", for a refactor)
   - Testing steps
   - Screenshots / Output — pass the runtime-verification output kept in
     Step 2 (from the final run, if the gate re-ran in Step 3) through to
     `pr-description`, which picks the form (screenshots, trimmed output,
     or `N/A — <reason>`) and trims and redacts per its Screenshots /
     Output Guidance. A PASS verdict in the risk table is not enough on
     its own for a non-UI runnable change. Never invent output.
   - Notes for reviewers
   - Checklist

If the user provided a title/description override, use it instead of
generating one.

```bash
# Write the description to a file first (e.g., with the Write tool), then
# pass the file. Never put the body inside a double-quoted shell argument:
# backticks in code fences and `$VAR` / `$(...)` in pasted commands or
# output would be run or expanded, which can leak real secrets into the PR.
# Write the file outside the repo (a temp path) and delete it afterwards.
# Single-quote the title so backticked names in it aren't run; write
# any ' inside it as '\''.
gh pr create --title '<title>' --body-file <description-file> --base <confirmed-base>

# Or if gh isn't available, provide the URL
echo "Create PR at: https://github.com/$(git remote get-url origin | sed 's/.*github.com[:/]//' | sed 's/.git$//')/compare/<confirmed-base>...<branch>"
```

Pass the description with `--body-file`, never inline as
`--body "<description>"`: descriptions are markdown full of backticks, and
diffs, gate findings, or runtime output can put `$(...)` or `$VAR` in them,
which a double-quoted argument would execute or expand. A file (rather than
a heredoc piped to `gh`) also lets the `pr-description` secret scan run
against exactly what gets sent.

If the branch already has an open PR, push without creating a new one and
update the PR description's risk assessment and Screenshots / Output
with the latest gate results and runtime output. Re-apply the
`pr-description` redaction rule and scan to the updated body, then run
`gh pr edit <number> --body-file <description-file>` (same rule).

Task status: before `gh pr create`, make sure the active dev-tasks
folder's `## Release Steps` lists anything the release will need (deploy
pairings, migrations, flags), or `None`. Once the PR exists, set its
status line to `Status: PR open (#N)` and append the transition to its
`context.md` Session Log.

### Step 6: Report

```
## Shipped

**Branch:** <branch> → <base>
**Risk:** <LOW / MEDIUM / HIGH / CRITICAL> (<one-line justification>)
**Gates:** <n> ran clean, <n> fixed then clean, <n> not run, <n> N/A
**Simplify:** <Ran — <n> commits / Ran — no changes / Skipped — <reason> / N/A>
**Commits added by ship mode:** <simplify and fix commits, or none>
**Fixed below threshold:** <n contained LOW/MEDIUM findings fixed in this PR>
**Unfixed introduced:** <n> (listed under Unresolved findings in the PR; offer an issue for each not-contained one)
**Existing issues:** <n> reported, <n> linked or filed
**Reported privately:** <existing security findings and unfixed introduced security findings (file:line, class, severity), and security re-triage reasons; recommend a security advisory>
**Dropped candidates:** <unreproduced /code-review findings; security-auditor drops (see its report)>
**PR:** <url or instructions>
**Files:** <count> changed, <insertions> insertions, <deletions> deletions
```

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
