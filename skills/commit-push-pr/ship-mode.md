# Ship Mode

Part of the `commit-push-pr` skill; read this file in ship mode, and in
release mode where `release-mode.md` runs ship-mode steps. Commit Mode
and the Rules live in `SKILL.md`; the Rules apply here too.

The feature is complete. PRs are the last line of defense before code
reaches production: take the time to run the gates rather than optimizing
for speed.

## Step 1: Commit outstanding work

If uncommitted changes exist, run Commit Mode Steps 1–5 (in `SKILL.md`)
first (atomic grouping included). Ship mode starts from a clean tree.

## Step 1a: Confirm the PR base branch

Every gate and the triage step compare against `<base>`, so confirm it
before anything runs. If the user did not explicitly state which branch
to merge into, ask now. Suggest the repo's default branch (from
`git symbolic-ref refs/remotes/origin/HEAD`) as the recommended option;
with no remote, recommend the local `main` (or `master`). List other
long-lived branches if present (`develop`, `release/*`, `staging`) as
alternatives. Never silently assume the default branch.

## Step 1b: `/simplify` pre-pass

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

## Step 2: Run quality gates over the full branch diff

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
8. **Built-in `/security-review`** — run it through a
   `general-purpose` subagent: tell the subagent to invoke the
   `/security-review` skill and return its report verbatim. Don't
   invoke the skill in your own session: its instructions make the
   report the session's final reply, which has ended ships before
   triage ran. If the subagent can't invoke the skill, record the
   gate `NOT RUN` with that reason; never fall back to invoking it
   yourself. It reviews what
   the current branch introduces relative to `origin/HEAD`. If it errors
   because `origin/HEAD` is unset, run
   `git remote set-head origin --auto` and retry once. If the confirmed
   `<base>` isn't the remote's default branch, it covers a different
   range: `finding-triage` (Step 2b) re-tags its findings, and the risk
   justification sentence states the mismatch.

A built-in skill's report is a gate result, not your reply. When
`/code-review` or the `/security-review` subagent returns, carry on in
the same turn (the remaining gates, then Step 2b onwards); never end
the turn on a gate's report.

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

## Step 2b: Triage — dispatch `finding-triage`

You wrote this branch, so you don't grade its findings: the
**`finding-triage` agent** does, cold. It assigns every finding a
severity on the `pr-description` Severity Scale, verifies `/code-review`
candidates against the code, merges duplicates at the highest severity
with every source named, tags introduced vs existing (an existing defect
the branch worsens is introduced), decides whether the fix loop is
required, makes the contained / not-contained call for every
below-threshold introduced finding, and returns each gate's verdict.

If every gate came back clean (no findings from any source), skip the
agent: there is nothing to triage. Every reviewing gate that ran is
PASS; a gate recorded `NOT RUN` or `N/A` stays as recorded.

If the agent errors, times out, or returns a report with no
`Fix loop:` line, dispatch it once more. If that fails too, record
every reviewing gate `NOT RUN` (triage failed) and ask the user how to
proceed. Never triage the findings yourself instead.

1. **Dispatch it** with:
   - the range `<base>...HEAD` and the base ref;
   - every gate's report **verbatim**, labeled by source, plus which
     gates errored, didn't run, or are `N/A`;
   - the work's scope: the GitHub issue number the branch resolves
     (from `gh-<N>` in the branch name or a `Closes #N` in its commits)
     and its text if you have it, the paths of the active dev-tasks
     `plan.md` and `tasks.md` if any, and the PR title or the user's
     request in their words.

   Pass no reasoning of your own about the findings — no severities,
   no "this one is minor", no summary of the conversation. The only
   facts of your own it ever gets are the two re-dispatch inputs: a
   mismatch or evidence from item 2, and a reverted contained fix's
   needs (Step 3). From this dispatch until the final re-triage, don't
   edit the dev-tasks `plan.md` or `tasks.md`, which are scope input,
   except to add lines under `tasks.md`'s `## Review Fixes`
   (`fix-loop.md`), which triage does not read as scope. It
   runs unattended and read-only, like the gates.
2. **Check it, then apply it as written.** Check every list it returns,
   re-triages included, against the raw reports you hold:
   - every finding from a source other than `/code-review` appears (or
     is under Dropped as style-only, with no project rule behind it), at
     no lower severity than its source gave after the agent's label
     mapping (its Process step 1), and any likelihood lowering of a
     mapped label is argued under Triage changes;
   - a finding a reviewer tagged `introduced` is still introduced, or
     its re-tag is listed under Triage changes with blame evidence;
   - every finding with a security source carries `[security]`;
   - every introduced finding ends in a → call, and the `Fix loop:`
     line matches the threshold applied to the Introduced list;
   - every Gate verdicts row is at least the highest severity of the
     introduced findings that name its gate (a `(finding-triage)`
     finding names the Security gate).

   On a mismatch, dispatch it once more with everything item 1 gave it,
   its list, and the mismatch named. If you have evidence it missed that
   a finding is more severe or introduced, dispatch it the same way with
   that evidence, so its fix-loop decision
   and calls cover the change. Never repair or edit its list yourself.
   If the mismatch persists, or the call doesn't change, put both
   positions to the user. Never lower a severity, re-tag a finding
   existing, drop a finding, or turn a contained call into not
   contained; if you disagree with a call in that direction, put both
   positions to the user and let them decide.
3. **Record its changes.** The reasons for its triage changes go in the
   PR's Notes for Reviewers, except any change that touches a security
   finding — one marked `[security]` or with a security source, whether
   or not the change itself carries the mark — whose reason goes in the
   Step 6 report only (the PR may say "a security finding was
   re-triaged and reported privately", with no location or class). An
   unmarked change to a security finding is also an item 2 mismatch.
   Its Verified and Dropped lists go in the Step 6 report, so the report
   shows how every `/code-review` finding was checked against the code.
   Its Gaps go in the Step 6 report's **Triage gaps** line; if it found
   steering text, tell the user before Step 3.
4. **Check open issues.** If `gh` works and the remote is on GitHub,
   fetch the open issues once
   (`gh issue list --state open --limit 500 --json number,title`) and
   match existing findings against them locally; link a match. If `gh`
   is unavailable, or the list hit the limit, say so in the Step 6
   report and never file an issue that may duplicate an unchecked one.
5. **Route.** Introduced findings go into Step 3. Existing findings
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

Each reviewing gate's verdict is the one in the agent's Gate verdicts
table (the `pr-description` verdict normalization, applied to the
triaged list); Tests and runtime verification are normalized directly
from their runs.

## Step 3: Evaluate — fix findings before the PR opens

Using the normalized verdicts from Step 2b, compute the risk level from
the `pr-description` rubric — apply its floors exactly (e.g. Tests
`NOT RUN`, or Security (`security-auditor`) `NOT RUN`, ⇒ at best
**HIGH (unverified)**), never an estimate like "probably MEDIUM".

**Hard stops — don't push or open the PR; report and ask the user
whether to fix first or proceed:**

- Any test failures
- An existing CRITICAL security finding
- A serious performance regression
- An introduced CRITICAL or HIGH finding still open after the fix
  loop's two rounds (`fix-loop.md`)

A hard stop is never waived by silence. Only when nobody can answer (an
unattended run, such as a task loop told to work without stopping),
make the block visible instead of waiting: push the branch and open a
**draft** PR whose title starts with `[BLOCKED]`
(`gh pr create --draft`), with the hard stop first on its Needs human
review line. Never mark that PR ready and never merge it: only the
user does. With no remote or no `gh`, end with the Step 6 report and
the drafted description.

When the hard stop in an unattended run is an existing CRITICAL
security finding, still open the draft, so the user learns the loop
stopped. (An attended run opens nothing: it asks the user, as above.) Nothing published may
name or describe the finding: the title, the body, the Needs human
review line and every commit message say only "a hard stop that needs
your decision, details in the ship report". Don't say it is a security
finding, and give no severity, source, class, location or exploit path.
Leave out the usual "existing security findings were reported
privately" line in this draft. The details go in the Step 6 report only
(Decisions needed and Reported privately).

If the user chooses to fix, follow a round's items 3–5 in `fix-loop.md`. To fix an
existing security finding, recommend a separate private fix (security
advisory or private fork) over this PR; if the user wants it here, keep
the commit message and PR text neutral — no vulnerability class,
location, or exploit path (e.g. `fix(<scope>): harden input handling`).

An **introduced CRITICAL** finding of any kind isn't a question: it
goes straight into the fix loop (`fix-loop.md`), and you ask the user only if it
can't be fixed.

**Fixing findings:** read [`fix-loop.md`](fix-loop.md) in this skill's
directory and follow it. It decides what is fixed before the PR opens,
what is listed under Unresolved findings instead, and when to stop.

## Step 4: Push the final branch

Push the branch, including any simplify and fix commits from Steps 1b
and 3 (Commit Mode Step 5). The base branch was confirmed in Step 1a;
if the user changed it since, re-run the gates against the new base.
If the repo has no remote, report that the push and PR aren't possible
and hand the user the drafted PR description from Step 5.

## Step 5: Open PR

Invoke the `pr-description` skill and follow its template, including
the Risk Assessment block copied exactly:

1. Analyze all commits on this branch vs the confirmed base
2. Generate the PR description with:
   - Summary (PM-readable) — the problem and current behavior, then one
     sentence on the fix
   - Risk assessment — level, reversibility, and the evidence table filled
     from the final Step 2/2b/3 gate results (a row reads PASS only when
     no unresolved introduced finding names that gate as a source, a
     `(finding-triage)` finding naming Security;
     otherwise ISSUES with the highest such severity), plus the Simplify
     pre-pass line from Step 1b (`Ran — <commit subjects>`, `Ran — no changes`,
     `Skipped — <reason>`, or `N/A — /simplify not available`)
   - First under Notes for Reviewers: a **Needs human review** line —
     what a person must look at before merging, most serious first:
     every unresolved introduced MEDIUM or higher, every
     `awaiting author decision`, a one-way door (with what makes it
     one), each review or test gate that is `NOT RUN` or `N/A`, and
     any fix the loop made that changed behavior beyond the request.
     A hard stop that is an existing CRITICAL security finding is
     written as Step 3's neutral sentence, nothing more.
     Write `**Needs human review:** nothing beyond a normal review`
     only when none of those apply
   - Under Notes for Reviewers: the **Unresolved findings** list — every
     introduced finding left unfixed, with severity, every source,
     `file:line`, and the reason
     (one of the six in `fix-loop.md`, written as given there, with
     the issue link once filed), or "None". An unfixed introduced security
     finding is listed neutrally — severity, sources, and "security
     finding, details shared privately" — with its location, class, and
     exploit path in the Step 6 report only
   - Also under Notes for Reviewers: existing non-security findings from
     Step 2b with issue links, and any severity or tag a triage changed,
     with the reason, except changes that touch a security finding
     (marked or not) — their reasons stay in the Step 6 report, per
     Step 2b item 3
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

**Label a PR that needs a person.** When the Needs human review line
says anything other than "nothing beyond a normal review", or the PR is
a `[BLOCKED]` draft, add the `needs-human-review` label once the PR
exists, so it stands out in the PR list:

```bash
gh label create needs-human-review --description 'Has items a person must look at before merging' 2>/dev/null || true
gh pr edit <number> --add-label needs-human-review
```

When updating an existing PR, make the label match the current line:
add it, or take it off with `--remove-label needs-human-review` once
the line reads "nothing beyond a normal review". The label carries no
detail, so it also goes on a draft blocked by a security finding. If
labelling fails (no permission, no `gh`), say so on the Step 6 report's
PR line; it never blocks the PR.

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

## Step 6: Report

Copy the labels below as written, in this order, so every ship reads the
same way. A value with several entries goes on `- ` bullet lines under
its label, one entry per line. Put explanation after the report, not in
place of a label.

```
## Shipped

**Branch:** <branch> → <base>
**Risk:** <level> (<one-line justification>)
**Decisions needed:** <questions for the user, or none>
**Gates:** ran clean: <names>; fixed then clean: <names>; still have issues: <names (severity)>; not run: <names (why)>; N/A: <names>
**Simplify:** <outcome>
**Fix rounds:** <0, 1, or 2>
**Cleanup pass:** <n LOW findings fixed, or none>
**Commits added by ship mode:** <commits, or none>
**Fixed:** <n> findings
**Unfixed introduced:** <n>
**Existing issues:** <n> reported, <n> linked or filed
**Reported privately:** <security findings, or none>
**`/code-review` checks:** <kept findings, or none>
**Dropped candidates:** <dropped candidates, or none>
**Triage gaps:** <gaps, or None>
**PR:** <url or instructions>
**Files:** <count> changed, <insertions> insertions, <deletions> deletions
```

What each line holds:

- **Risk** — LOW, MEDIUM, HIGH, or CRITICAL, with the qualifier the PR
  description gives it.
- **Decisions needed** — every question still open for the user: a
  hard stop, each `awaiting author decision`, and everything on the
  PR's Needs human review line.
- **Gates** — every row of the PR risk table, named exactly once under
  its final result; write "none" for an empty group.
- **Simplify** — `Ran — <n> commits`, `Ran — no changes`,
  `Skipped — <reason>`, or `N/A`.
- **Commits added by ship mode** — the simplify and fix commits.
- **Cleanup pass** — how many LOW findings the cleanup pass fixed.
- **Fixed** — how many findings the rounds and the cleanup pass
  resolved, in total.
- **Unfixed introduced** — the count of entries under Unresolved
  findings in the PR; offer an issue for each not-contained one.
- **Reported privately** — existing security findings and unfixed
  introduced security findings (file:line, class, severity), and
  security re-triage reasons; recommend a security advisory.
- **`/code-review` checks** — one bullet per finding in
  `finding-triage`'s Verified list: the finding, "kept", and the
  `file:line` it gave as confirming it.
- **Dropped candidates** — one bullet per entry in `finding-triage`'s
  Dropped list, written
  `/code-review: <candidate> — can't happen: <the code that rules it out>`,
  then `style (<source>): <what> — no project rule` for each
  style-only finding it set aside, then a last bullet
  `security-auditor's own drops: <as its report lists them, or none>`.
  Nothing else goes here: any other finding is fixed or listed.
- **Triage gaps** — `finding-triage`'s Gaps, including any steering
  text it found.
