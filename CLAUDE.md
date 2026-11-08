# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` — the
  plugin manifest and the marketplace catalog. Both carry a `version`
  field that must stay in sync.

Components reference each other by name: `commit-push-pr` invokes the
`commit-message` and `pr-description` skills, the `code-reviewer`,
`security-auditor`, `perf-audit`, and `doc-sync` agents, and the
built-in Claude Code skills `/code-review` and `/security-review` (plus
a `/simplify` pre-pass on code changes); `pr-description`'s risk rubric is
consumed by `commit-push-pr` ship mode, whose triage assigns severities
to `/code-review` findings using `pr-description`'s Severity Scale —
which the `code-reviewer` agent mirrors, so change both together. When renaming or
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
- **Dev-tasks separation.** `.claude/dev/` task files are internal
  tracking state — gitignored, never staged or committed, invisible to
  other contributors. Commit messages and PR descriptions carry
  zero trace of them — no paths, no dev-tasks task IDs, no bookkeeping
  mentions. External tracker IDs (Jira keys, GitHub issues) are fine.
- **Atomic commits.** One commit = one task/logical change; never bundle
  unrelated changes.
- **Use every available reviewer.** Ship mode runs the toolkit's review
  agents and the built-in `/code-review` and `/security-review` skills
  side by side as gates over `<base>...HEAD`; a triage step assigns
  severities to `/code-review` findings, merges duplicates (highest
  severity, every source named), and tags introduced vs existing, so the
  pr-description rubric gets one verdict. Side-by-side runs on a real PR
  showed each side catches what the other misses: the agents bring
  installed tooling, existing-code coverage, performance, docs, test
  coverage, and `.claude/rules/`; `/code-review` traces callers and
  audits removed behavior; `/security-review` filters false positives
  hard. Don't drop either side as redundant. A built-in skill missing
  from the running Claude Code version is `N/A`, which raises the risk
  level (never LOW). Never auto-run `/code-review ultra` — it is
  user-triggered and billed.
- **Sharpen is proactive but consent-gated.** The skill fires on
  agent- or user-detected mistakes and inefficiencies, but never modifies
  a harness file (CLAUDE.md, `.claude/rules/`, lint config, guard tests)
  without explicit user approval in that conversation.

## Working Tree vs Installed Plugin

The locally installed plugin (`claude-code-toolkit@rahul-claude-code-toolkit`)
is served from the marketplace clone under `~/.claude/plugins/`, which
pulls from GitHub — **edits in this working tree have no effect on live
sessions until committed, pushed, and the plugin is updated**. Don't test
component changes by invoking the installed skill/agent and expecting the
new behavior.

## Testing Skills — Eval Suite

Behavioral eval cases live under `evals/` (one dir per case: `prompt.md`
+ `graders/criteria.md`), run by `claude plugin eval` (Claude Code
2.1.269+). Run from the repo root:

```bash
# whole suite against the working tree (no push/reinstall needed)
claude plugin eval . --allow-tools Bash Write Edit
# one case, quick check
claude plugin eval . --case <case> --runs 1 --ablation none --allow-tools Bash Write Edit
```

`--allow-tools Bash Write Edit` is required: every case builds a scratch
git project, and a case's `allowed_tools` only grants read-only tools on
its own — gated tools (Bash, Write, Edit) also need the operator grant.
Each run is a full Claude session on your own credential and plan limits,
and the defaults (3 runs × with/without-plugin arms) multiply that, so
prefer `--case`, `--runs 1`, `--ablation none`, `--no-publish` and a
`--max-cost-usd` ceiling while iterating. Pass `--judge-model claude-sonnet-5-5`:
the default Haiku judge has failed correct runs that needed judgment.

Every case needs frontmatter or the runner rejects it. `prompt.md`
starts with `max_turns` and `allowed_tools` (listing Bash, Write and Edit
too); `graders/criteria.md` starts with `type: llm` and `weight: 1`. An
`llm` grader sees only the final message unless it sets `focus: trace`
(and then only the first and last 12 messages), so grade behavior —
commands run, files created, agents spawned — with `tool_used` /
`file_exists` graders. Copy an existing case's frontmatter;
`evals/README.md` has the exact block. Single runs are noisy; confirm a
change at the default 3 runs.

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
