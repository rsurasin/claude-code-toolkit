# Eval Suite

Behavioral eval cases for this plugin's skills, in the `claude plugin eval`
layout (`evals/<case>/prompt.md` + `evals/<case>/graders/criteria.md`).

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
> changing the corresponding skills.

## Cases

| Case | Skill under test | Asserts |
|------|------------------|---------|
| `commit-mode-no-gates` | commit-push-pr | Intermediate commits run no gate agents and open no PR |
| `commit-mode-atomic-split` | commit-push-pr | Multi-task working trees become separate atomic commits |
| `ship-mode-base-branch` | commit-push-pr | Ship mode asks for the PR base branch when unstated |
| `commit-message-no-devtasks` | commit-message | Commit messages carry zero trace of dev-tasks state |
| `sharpen-consent-gate` | sharpen | Self-detected mistakes are proposed as harness rules, never applied without consent |
| `review-plan-critical-rubric` | review-plan | Irreversible + flawed plans rate CRITICAL from a fresh-context subagent, verdict consistent with level |
