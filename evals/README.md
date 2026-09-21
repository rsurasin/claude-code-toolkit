# Eval Suite

Behavioral eval cases for this plugin's skills and agents, in the
`claude plugin eval` layout (`evals/<case>/prompt.md` +
`evals/<case>/graders/criteria.md`).

Evals cover agents as well as skills: each case runs plugin-equipped
Claude against a user-style prompt, so agent cases test both *dispatch*
(the right agent gets spawned, not inline work or a native-skill
substitute) and the agent's *contract* (its output holds its promises).

> `claude plugin eval` is in early access. Once enabled, run against the
> working tree from the repo root:
>
> ```bash
> claude plugin eval .
> ```
>
> Or against the installed plugin (adds a no-plugin ablation baseline):
>
> ```bash
> claude plugin eval claude-code-toolkit@rahul-claude-code-toolkit
> ```
>
> Until then, the criteria files double as manual review checklists when
> changing the corresponding skills and agents.

## Skill Cases

| Case | Skill under test | Asserts |
|------|------------------|---------|
| `commit-mode-no-gates` | commit-push-pr | Intermediate commits run no gate agents and open no PR |
| `commit-mode-atomic-split` | commit-push-pr | Multi-task working trees become separate atomic commits |
| `ship-mode-base-branch` | commit-push-pr | Ship mode asks for the PR base branch when unstated |
| `commit-message-no-devtasks` | commit-message | Commit messages carry zero trace of dev-tasks state |
| `commit-message-breaking-change` | commit-message | Breaking API changes get `!` and a `BREAKING CHANGE:` footer |
| `pr-description-evidence-floors` | pr-description | Unverified dimensions are NOT RUN and floor the risk at HIGH (unverified) |
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
