# Eval Suite

Behavioral eval cases for this plugin's skills and agents, in the
`claude plugin eval` layout (`evals/<case>/prompt.md` +
`evals/<case>/graders/criteria.md`).

Evals cover agents as well as skills: each case runs plugin-equipped
Claude against a user-style prompt, so agent cases test both *dispatch*
(the right agent gets spawned, not inline work or a native-skill
substitute) and the agent's *contract* (its output holds its promises).

## Running

Requires Claude Code 2.1.269 or later (`claude update` if the command
reports early access). Run from the repo root:

```bash
# Whole suite against the working tree (no push or reinstall needed)
claude plugin eval . --allow-tools Bash Write Edit

# One case, one run per arm, no no-plugin baseline: a quick check
claude plugin eval . --case pr-description-non-ui-output --runs 1 --ablation none --allow-tools Bash Write Edit

# Against the installed plugin instead of the working tree
claude plugin eval claude-code-toolkit@rahul-claude-code-toolkit --allow-tools Bash Write Edit
```

`--allow-tools Bash Write Edit` is required: every case builds a scratch
git project, and a case's `allowed_tools` only grants read-only tools on
its own. Gated tools (Bash, Write, Edit) also need this operator grant,
or the runs fail.

Each run is a full Claude session on your own credential and plan
limits. Both targets run a second, no-plugin baseline arm by default, so the
report shows what the plugin adds; `--ablation none` skips it.

Useful options:

- `--case <glob>` — run only matching cases.
- `--runs <n>` — runs per arm (default 3). Single runs are noisy; confirm
  a change at 3.
- `-j <n>` — run cases in parallel (1–8).
- `--json [file]`, `--output-dir <dir>`, `--report <path>` — save results.
- `--max-cost-usd <usd>` — stop at a cost ceiling.
- `--trust-plugin` — skip the first-run trust prompt (for CI).
- `--judge-model claude-sonnet-5-5` — the default Haiku judge has failed
  correct runs that need judgment (e.g. a sandbox refusal the criteria
  excuse); a Sonnet judge costs cents per case.
- `--no-publish` — keep the HTML report local (it publishes to claude.ai
  by default when your account supports it).

While iterating, keep runs cheap: `--case`, `--runs 1`, `--ablation none`,
`--no-publish`, and a `--max-cost-usd` ceiling.

If a Bash-granting run fails with "the Docker credential store … holds a
symbolic link", the sandbox refuses to run while `~/.docker` contains
symlinks (common on WSL with Docker Desktop). Move those links out of
`~/.docker` for the duration of the run and restore them afterwards.

The criteria files also work as manual review checklists when changing
the corresponding skills and agents.

## Case Format

Each case is a directory with a `prompt.md` and a `graders/criteria.md`.
Both files need frontmatter; the runner rejects a case without it.

`prompt.md`: every case builds a scratch git project, so it grants the
tools for that:

```markdown
---
max_turns: 30
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---
<user-style prompt>
```

`graders/criteria.md`: an LLM-judged checklist:

```markdown
---
type: llm
weight: 1
focus: trace   # default last_message: the judge sees only the final reply
---
<PASS / FAIL criteria>
```

With `focus: trace` the judge still sees only the first and last 12
messages, so check behavior — commands run, files created, agents
spawned — with deterministic graders beside it (`graders/<name>.md`,
frontmatter only):

```markdown
---
type: tool_used          # also: file_exists, regex, tool_order
tool: Bash
input_match: 'gh pr create'
min: 0
max: 0                   # never called
---
```

Match `tool_used` on tool input rather than `regex` over the trace: the
trace includes the loaded skill text, which names the very commands you
are checking for.

Add a row for each new case to the tables below.

## Skill Cases

| Case | Skill under test | Asserts |
|------|------------------|---------|
| `commit-mode-no-gates` | commit-push-pr | Intermediate commits run no gate agents and open no PR |
| `commit-mode-atomic-split` | commit-push-pr | Multi-task working trees become separate atomic commits |
| `ship-mode-base-branch` | commit-push-pr | Ship mode asks for the PR base branch when unstated, before any gate runs |
| `ship-mode-builtin-gates` | commit-push-pr | Ship mode runs `/code-review` and `/security-review` alongside the agents, triages their findings into one risk table, commits a fix for the introduced race before re-running gates, keeps `N/A` gates above LOW, reports the `/simplify` pre-pass outcome, and lists unfixed introduced findings under Unresolved findings |
| `release-mode-closeout` | commit-push-pr, dev-tasks | The release PR lists every feature, its release steps, and a `Closes #N` per issue, with no `.claude/dev/` trace; folders stay unreleased until merge |
| `release-mode-post-merge` | commit-push-pr, dev-tasks | After the release merges: issues confirmed (or reported unconfirmed), one-way release steps not run without approval, folders marked `released (#N, <date>)` |
| `commit-message-no-devtasks` | commit-message | Commit messages carry zero trace of dev-tasks state |
| `commit-message-breaking-change` | commit-message | Breaking API changes get `!` and a `BREAKING CHANGE:` footer |
| `pr-description-evidence-floors` | pr-description | Unverified dimensions are NOT RUN and floor the risk at HIGH (unverified) |
| `pr-description-behavior-before-after` | pr-description | Summary states the problem and current behavior, Changes leads with the new behavior, and a refactor says "no intended behavior change" (verified or unverified) in both |
| `pr-description-non-ui-output` | pr-description | A non-UI change's Screenshots / Output shows the command, exit code, and trimmed output, with the token redacted |
| `dev-tasks-structure` | dev-tasks | Tasks land in `.claude/dev/<category>/<UTC-date>-<slug>/` with all three files, gitignored |
| `issue-number-naming` | dev-tasks + commit-push-pr | Work from GitHub issue #N gets a `<date>-gh-<N>-<slug>` folder and a `<type>/gh-<N>-<slug>` branch |
| `pr-description-issue-link` | pr-description | A `gh-<N>-` branch confirmed by the user yields `Closes #N` in the body; branch-only evidence yields `Refs #N` |
| `dev-tasks-status-transitions` | dev-tasks | A reported merge sets the fixed `merged to <base> (<sha>), not released` status; session start lists folders awaiting release |
| `dev-tasks-decision-record` | dev-tasks | A product question settled in conversation is recorded before moving on, with both options, pros and cons, the reason, and the trade-off accepted |
| `sharpen-consent-gate` | sharpen | Self-detected mistakes are proposed as harness rules, never applied without consent |
| `review-plan-critical-rubric` | review-plan | Irreversible + flawed plans rate CRITICAL from a fresh-context subagent, verdict consistent with level |

## Agent Cases

| Case | Agent under test | Asserts |
|------|------------------|---------|
| `code-reviewer-finds-planted-bug` | code-reviewer | Dispatches as a dedicated agent, finds a planted off-by-one with file:line, defined severity, `introduced` tag, and a failure scenario |
| `security-auditor-detects-injection` | security-auditor | Flags planted SQL injection at CRITICAL/HIGH with confidence ≥ 8/10 and an `existing` tag, recommends parameterized queries |
| `claude-sync-approval-gate` | claude-sync | Reports drift before editing, edits only with approval, never touches never-rules or skills |
| `doc-sync-scope` | doc-sync | Fixes repo docs, leaves `.claude/` config to claude-sync, never invents or deletes docs |
