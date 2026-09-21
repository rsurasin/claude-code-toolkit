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
`commit-message` and `pr-description` skills and the `code-reviewer`,
`security-auditor`, `perf-audit`, and `doc-sync` agents; `pr-description`'s
risk rubric is consumed by `commit-push-pr` ship mode. When renaming or
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
- **Quality gates are dedicated agents, not skills — independence over
  nativeness.** Native code-review / security-review skills run in the
  context of the agent that wrote the code (self-review bias) and their
  output formats vary by Claude Code version; the toolkit's spawned
  agents review cold and return verdicts the pr-description rubric can
  normalize. Native skills are user-facing complements, never gate
  replacements.
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
+ `graders/criteria.md`), in the `claude plugin eval` layout. That
command is currently **early access** (`claude plugin eval init` exits 1);
until it's enabled, use the criteria files as manual review checklists
when changing the corresponding skill. Once enabled:

```bash
claude plugin eval .    # eval the working tree (no push/reinstall needed)
```

When changing a skill's contract (modes, thresholds, separation rules),
update or add the matching eval case in the same commit.

## Releasing a New Version

1. Bump `version` in **both** `.claude-plugin/plugin.json` and
   `.claude-plugin/marketplace.json` (major bump for removed/renamed
   components). The version bump should be the release's final commit.
2. The plugin `source` in marketplace.json must stay an explicit https
   git URL (`{"source": "git", "url": "https://github.com/..."}`) — the
   shorter `github` source type clones over SSH and breaks
   `claude plugin update` in shells without a loaded SSH key.
3. Validate the manifests parse:
   `node -e "require('./.claude-plugin/plugin.json'); require('./.claude-plugin/marketplace.json')"`
4. Commit and push to `main`.
5. Update the installed copy:
   `claude plugin update claude-code-toolkit@rahul-claude-code-toolkit`
   (restart Claude Code to apply; if the new version isn't found, run
   `claude plugin marketplace update rahul-claude-code-toolkit` first).
