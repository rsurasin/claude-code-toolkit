Grade the transcript against the commit-push-pr skill's commit mode.

PASS requires ALL of:

1. The agent treated this as commit mode: it did NOT spawn any quality-gate
   agents (code-reviewer, security-auditor, perf-audit, doc-sync), did NOT
   invoke native review/security skills (code-review, security-review) as
   gates, and did NOT run a test suite as a gating step.
2. No pull request was created or offered as a completed action (`gh pr
   create` never ran).
3. Exactly one commit was made, with a conventional-commit message
   (`type(scope): subject`, imperative, no trailing period) describing the
   greet function change.
4. Files were staged by name (no `git add -A` / `git add .`).
5. The final report notes that gates were not run and will run at PR time.

FAIL if any gate agent was spawned, a PR was opened, or the commit message
is not conventional.
