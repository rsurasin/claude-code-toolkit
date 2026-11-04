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
claude plugin eval .

# One case, one run per arm, no no-plugin baseline: a quick check
claude plugin eval . --case pr-description-non-ui-output --runs 1 --ablation none

# Against the installed plugin instead of the working tree
claude plugin eval claude-code-toolkit@rahul-claude-code-toolkit
```

Both targets run a second, no-plugin baseline arm by default, so the
report shows what the plugin adds; `--ablation none` skips it.

Useful options:

- `--allow-tools Bash Write Edit` — most cases build a scratch git
  project, so they need these tools.
- `--case <glob>` — run only matching cases.
- `--runs <n>` — runs per arm (default 3). Single runs are noisy; confirm
  a change at 3.
- `-j <n>` — run cases in parallel (1–8).
- `--json [file]`, `--output-dir <dir>`, `--report <path>` — save results.
- `--max-cost-usd <usd>` — stop at a cost ceiling.
- `--trust-plugin` — skip the first-run trust prompt (for CI).

The criteria files also work as manual review checklists when changing
the corresponding skills and agents.

## Skill Cases

| Case | Skill under test | Asserts |
|------|------------------|---------|
| `commit-mode-no-gates` | commit-push-pr | Intermediate commits run no gate agents and open no PR |
| `commit-mode-atomic-split` | commit-push-pr | Multi-task working trees become separate atomic commits |
| `ship-mode-base-branch` | commit-push-pr | Ship mode asks for the PR base branch when unstated |
| `commit-message-no-devtasks` | commit-message | Commit messages carry zero trace of dev-tasks state |
| `commit-message-breaking-change` | commit-message | Breaking API changes get `!` and a `BREAKING CHANGE:` footer |
| `pr-description-evidence-floors` | pr-description | Unverified dimensions are NOT RUN and floor the risk at HIGH (unverified) |
| `pr-description-behavior-before-after` | pr-description | Summary states the problem and current behavior, Changes leads with the new behavior, and a refactor says "no behavior change" in both |
| `pr-description-non-ui-output` | pr-description | A non-UI change's Screenshots / Output shows the command, exit code, and trimmed output, with the token redacted |
| `dev-tasks-structure` | dev-tasks | Tasks land in `.claude/dev/<category>/<UTC-date>-<slug>/` with all three files, gitignored |
| `sharpen-consent-gate` | sharpen | Self-detected mistakes are proposed as harness rules, never applied without consent |
| `review-plan-critical-rubric` | review-plan | Irreversible + flawed plans rate CRITICAL from a fresh-context subagent, verdict consistent with level |

## Agent Cases

| Case | Agent under test | Asserts |
|------|------------------|---------|
| `code-reviewer-finds-planted-bug` | code-reviewer | Dispatches as a dedicated agent, finds a planted off-by-one with file:line and severity |
| `security-auditor-detects-injection` | security-auditor | Flags planted SQL injection at CRITICAL/HIGH, recommends parameterized queries |
| `claude-sync-approval-gate` | claude-sync | Reports drift before editing, edits only with approval, never touches never-rules or skills |
| `doc-sync-scope` | doc-sync | Fixes repo docs, leaves `.claude/` config to claude-sync, never invents or deletes docs |
