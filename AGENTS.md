# AGENTS.md

This file provides guidance to any coding agent working with code in this
repository.

## What This Repo Is

A Claude Code plugin distributed via its own marketplace (this GitHub repo).
It contains no application code — only markdown component definitions and
two JSON manifests. There is no build, test suite, or linter; verification
means checking that the JSON manifests parse and the markdown is internally
consistent.

## Structure and How Components Relate

- `agents/*.md` — subagent definitions (frontmatter + system prompt),
  auto-dispatched by request.
- `skills/<name>/SKILL.md` — model-invoked skills, also invocable as
  `/<name>`. `$ARGUMENTS` substitution does NOT work in SKILL.md files
  (it's a command-template feature) — skills must phrase argument
  handling as "if the user provided X in their request". The repo no
  longer ships a `commands/` directory; all former commands are skills.
  `commit-push-pr` is split across files in its directory: `SKILL.md`
  (commit mode and the Rules), `ship-mode.md` (gates, triage, PR),
  `fix-loop.md` (what is fixed before the PR opens), and
  `release-mode.md`.
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` — the
  plugin manifest and the marketplace catalog. Both carry a `version`
  field that must stay in sync.

Components reference each other by name: `commit-push-pr` invokes the
`commit-message` and `pr-description` skills, the `code-reviewer`,
`security-auditor`, `perf-audit`, `doc-sync`, and `finding-triage`
agents, and the built-in Claude Code skills `/code-review` and `/security-review` (plus
a `/simplify` pre-pass on code changes); `pr-description`'s risk rubric is
consumed by `commit-push-pr` ship mode, whose `finding-triage` agent
assigns severities to every gate's findings using `pr-description`'s
Severity Scale — which `code-reviewer` and `finding-triage` both copy,
so change all three together. `doc-sync` leaves `.claude/` and every
agent instruction file to `agent-sync`.
`commit-push-pr` release mode also uses the `dev-tasks` skill's release
closeout to map merged PRs to task folders and mark them released. When renaming or
changing a component's behavior, grep for its name across `skills/`,
`agents/`, and `README.md` and update every reference — the README's
tables, workflow section, and directory tree mirror the components and
drift easily.

## Deliberate Design Decisions (do not undo)

- **No generic test/verify agents.** v3.0.0 removed `qa-runner` and
  `verify-app` on purpose: each consuming repo defines its own qa/verify
  skills in `.claude/`, which the `commit-push-pr` gates discover. Missing
  ones are recorded `NOT RUN`, raising the PR risk level. Don't re-add
  ecosystem-guessing agents.
- **Gates run at the PR boundary, not per commit.** `commit-push-pr` is
  two-mode: commit mode (fast, atomic, gate-free) and ship mode (all gates
  once over the full branch diff, findings fixed before the PR opens).
- **The fix loop is bounded and never hides a serious finding.**
  `fix-loop.md` allows at most two fix rounds (which re-run the
  reviewers) and then one cleanup pass for LOW findings. A closing
  review by `code-reviewer` and `security-auditor` reads only the
  cleanup commits and can only revert one, never start more fixing, so
  no commit ships unreviewed and the loop still ends. Unbounded rounds
  did not converge: every fix raised new minor findings. LOW findings
  are still fixed, not just listed. The hard stops come at two points.
  A test already failing on the base, or triage that failed twice
  (nothing graded the findings), stops the run before any fix
  round. A test the branch broke, or an introduced CRITICAL or HIGH,
  gets the two rounds first and is a hard stop only if still open (one
  the closing review raises outside the cleanup commits is a hard stop
  at once, since the rounds are over). On
  a hard stop no mergeable PR opens: an attended run asks, an
  unattended run opens a draft PR titled `[BLOCKED]`, and a user who
  chooses to proceed gets a draft without that title. Release mode has
  no rounds, so the same findings stop a release at once. Don't raise
  the round count or let a hard stop proceed on its own.
- **Existing problems don't block a PR.** An existing CRITICAL
  security finding, such as a vulnerable dependency the branch didn't
  touch, used to be a hard stop, which would block every ship until it
  was patched. The PR now opens as usual with a neutral Needs human
  review line, and the details stay in the ship report. A finding the
  branch causes or worsens (it changes the vulnerable lines, opens a
  new path to them, or adds, bumps or newly uses the vulnerable
  dependency) is tagged introduced and does block; `finding-triage`
  has the exact rule.
- **Unattended is defined, and doubt means attended.** A run is
  unattended only when it is headless or the author said up front to
  work without stopping to ask. `ship-mode.md` Step 3 lists what an
  unattended run does at each point that would ask; it never proceeds
  for the user, lowers a severity, or makes a one-way-door change.
- **The PR tells the human what to look at.** The goal is an unattended
  implement, review and fix loop where the human only merges. Every PR
  description carries a Needs human review line (unresolved MEDIUM or
  higher, pending decisions, one-way doors, gates that didn't run) and
  an Unresolved findings list with a reason per entry. A PR gets the
  `needs-human-review` label unless that line reads "nothing beyond a
  normal review", and the ship report opens with a one-line Status. The one-way doors are listed in
  `pr-description`'s Rules of evidence and put a PR at HIGH or above.
- **Style is the linter's job.** A style-only finding (formatting, line
  length, wording preference) is not a finding unless a project rule or
  linter config requires it. The Severity Scale says so in all three
  copies.
- **Review findings are tracked in the task files.** The fix loop
  writes an AUDIT entry in `context.md` per round and one line per
  finding under `## Review Fixes` in `tasks.md`. `finding-triage` does
  not read those as scope, so recording a finding cannot make it in
  scope; the rest of `plan.md` and `tasks.md` stays frozen during
  triage.
- **Tests are never weakened to pass a gate.** No deleting, skipping,
  or loosening a test; a test believed wrong is a question for the
  user.
- **Dev-tasks separation.** `.claude/dev/` task files are internal
  tracking state — gitignored, never staged or committed, invisible to
  other contributors. Commit messages and PR descriptions carry
  zero trace of them — no paths, no dev-tasks task IDs, no bookkeeping
  mentions. External tracker IDs (Jira keys, GitHub issues) are fine.
- **The author never grades its own findings.** Ship mode hands every
  gate's raw findings to the `finding-triage` agent, which assigns
  severities, tags introduced vs existing, decides the fix loop, and
  makes the contained / not-contained call with no context from the
  session that wrote the code. Ship mode applies its list as written:
  with evidence that a finding is more severe or introduced, or a list
  that fails its checks, it re-dispatches the agent and never edits the
  list, and a disagreement in the other direction goes to the user. If
  triage fails twice, that is a hard stop: the reviewing gates are
  `NOT RUN`, the PR lists the findings as `not triaged`, and the user
  decides; ship mode never triages inline. Don't move triage back into
  the authoring session.
- **Atomic commits.** One commit = one task/logical change; never bundle
  unrelated changes.
- **Use every available reviewer.** Ship mode runs the toolkit's review
  agents and the built-in `/code-review` and `/security-review` skills
  side by side as gates over `<base>...HEAD`; the `finding-triage`
  agent assigns severities to `/code-review` findings, merges duplicates
  (highest severity, every source named), and tags introduced vs
  existing, so the pr-description rubric gets one verdict. Side-by-side
  runs on a real PR showed each side catches what the other misses: the
  agents bring installed tooling, existing-code coverage, performance, docs, test
  coverage, and `.claude/rules/`; `/code-review` traces callers and
  audits removed behavior; `/security-review` filters false positives
  hard. Don't drop either side as redundant. A built-in skill missing
  from the running Claude Code version is `N/A`, which raises the risk
  level (never LOW). Never auto-run `/code-review ultra` — it is
  user-triggered and billed.
- **Sharpen is proactive but consent-gated.** The skill fires on
  agent- or user-detected mistakes and inefficiencies, but never modifies
  a harness file (CLAUDE.md or AGENTS.md, `.claude/rules/`, lint config,
  guard tests) without explicit user approval in that conversation. It
  writes rules to the file that holds the content, never through a
  symlink.

## Working Tree vs Installed Plugin

The locally installed plugin (`claude-code-toolkit@rahul-claude-code-toolkit`)
is served from the marketplace clone under `~/.claude/plugins/`, which
pulls from GitHub — **edits in this working tree have no effect on live
sessions until committed, pushed, and the plugin is updated**. Don't test
component changes by invoking the installed skill/agent and expecting the
new behavior.

## Testing Skills — Eval Suite

Behavioral eval cases live under `evals/` (one dir per case: `prompt.md`
+ `graders/*.md`), run by `claude plugin eval` (Claude Code
2.1.269+). Run from the repo root:

```bash
# whole suite against the working tree (no push/reinstall needed)
claude plugin eval . --allow-tools Bash Write Edit
# one case, quick check
claude plugin eval . --case <case> --runs 1 --ablation none --allow-tools Bash Write Edit
```

On Linux and WSL, Claude Code 2.1.304 through at least 2.1.322 needs
`TMPDIR` set outside `/tmp` for these commands, e.g.
`TMPDIR=$HOME/.cache/eval-tmp claude plugin eval . …` (create the
directory first): the eval sandbox denies writes to a scratch directory
under `/tmp`, so every case fails at `git init` with "Read-only file
system" (anthropics/claude-code#106687). Discard a run that shows it.

A run that fails because a write under `.claude/` was refused is not a
skill fault: the eval session runs in "don't ask" permission mode,
which denies some of those writes whatever `--allow-tools` grants
(path grants such as `'Edit(.claude/**)'` don't help). Cases that
write task files under `.claude/dev/` fail a run now and then for this
reason; `evals/README.md` says how to tell it apart.

`--allow-tools Bash Write Edit` is required: every case builds a scratch
git project, and a case's `allowed_tools` only grants read-only tools on
its own — gated tools (Bash, Write, Edit) also need the operator grant.
Each run is a full Claude session on your own credential and plan limits,
and the defaults (3 runs × with/without-plugin arms) multiply that, so
prefer `--case`, `--runs 1`, `--ablation none`, `--no-publish` and a
`--max-cost-usd` ceiling while iterating. Pass `--judge-model claude-opus-5-5`:
the default Haiku judge has failed correct runs that needed judgment,
and so has a Sonnet judge. Replaying three replies the Sonnet judge had
failed, it failed one of them every time and flipped on another, where
an Opus judge passed all three every time. The judge is a small part of
a run's cost.

Every case needs frontmatter or the runner rejects it. `prompt.md`
starts with `max_turns` and `allowed_tools` (listing Bash, Write and Edit
too); an LLM grader file (usually `graders/criteria.md`; several cases
split it into single-check files, such as `criteria-*.md` in
`finding-triage-security-marker` and `reply-*.md` in the ship-mode
and release-mode cases; `grep -rl 'type: llm' evals/*/graders` lists
them all) starts with
`type: llm` and `weight: 1`. An
`llm` grader sees only the final message unless it sets `focus: trace`
(and then only the first and last 12 messages), so grade behavior —
commands run, files created, agents spawned — with `tool_used` /
`file_exists` graders. Copy an existing case's frontmatter;
`evals/README.md` has the exact block. Single runs are noisy; confirm a
change at the default 3 runs.

Prefer deterministic graders. A `regex` grader can read the final reply
(`target: last_message`) or a file the agent wrote (`target:` with
`source: file` and `path:`), takes `flags: m`, and fails if the file is
missing; `ship-mode-builtin-gates` has the agent save the PR body to
`pr-body.md` for this. Give an LLM grader one check, not a list: the
judge has failed a multi-item rubric 3–0 on a reply that passed every
item alone, and it flips on identical text between runs. To find which
item fails without re-running a case, replay a saved reply: a scratch
plugin whose case prompt tells the model to repeat the reply verbatim,
with one grader per item (about $1).

When changing a skill's contract (modes, thresholds, separation rules),
update or add the matching eval case in the same commit.

## Releasing a New Version

1. Bump `version` in **both** `.claude-plugin/plugin.json` and
   `.claude-plugin/marketplace.json` (major bump for removed/renamed
   components). The version bump should be the release's final commit.
2. The plugin `source` in marketplace.json must stay the relative path
   `"./"` — the plugin lives in the marketplace repo, so it is served
   straight from the marketplace clone with no second clone. Do not use
   the `github` source type (clones over SSH, which breaks
   `claude plugin update` in shells without a loaded key) or the `git`
   type (not a supported plugin source type; only marketplaces accept
   it — plugins accept a relative path or `{"source": "url", ...}`).
3. Validate the manifests parse:
   `node -e "require('./.claude-plugin/plugin.json'); require('./.claude-plugin/marketplace.json')"`
4. Commit and push to `main`.
5. Update the installed copy — marketplace refresh first (the plugin is
   served from the marketplace clone, so this is what fetches the new
   version), then the plugin update, then restart Claude Code:
   `claude plugin marketplace update rahul-claude-code-toolkit`
   `claude plugin update claude-code-toolkit@rahul-claude-code-toolkit`
