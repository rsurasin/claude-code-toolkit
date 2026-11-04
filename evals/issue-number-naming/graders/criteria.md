---
type: llm
weight: 1
focus: last_message
---

Grade the agent's final reply against the GitHub-issue naming contract
shared by the dev-tasks and commit-push-pr skills. Separate graders check
that `date -u` ran, the branch name, and that no task file was staged.

Note: the eval sandbox may refuse writes under `.claude/`. If the reply
says so, that is the environment, not a failure — grade the folder the
agent attempted. Writing the files with a shell heredoc instead of the
Write tool is fine.

PASS requires ALL of:

1. The task folder is `.claude/dev/features/<YYYY-MM-DD>-gh-31-<slug>/` —
   the `gh-` prefix is present (not `<YYYY-MM-DD>-31-<slug>`) and the slug
   is kebab-case.
2. Because the work started on `main`, the commit landed on a feature
   branch named `<type>/gh-31-<slug>` (e.g. `feat/gh-31-password-reset`),
   not on `main`.
3. The task files were not committed, and the commit message the reply
   reports contains no `.claude/dev/` path or dev-tasks task ID.
   Referencing issue #31 (`Refs #31`, `Closes #31`) is allowed — it is an
   external tracker ID.

FAIL if the folder or branch lacks the issue number, the folder uses a
bare number without `gh-`, the commit went to `main`, or any dev-tasks
trace reached the commit.
