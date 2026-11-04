Grade the transcript against the GitHub-issue naming contract shared by the
dev-tasks and commit-push-pr skills.

PASS requires ALL of:

1. The task folder was created at
   `.claude/dev/features/<YYYY-MM-DD>-gh-31-<slug>/` — the `gh-` prefix
   is present (not `<YYYY-MM-DD>-31-<slug>`), the date came from `date -u`,
   and the slug is kebab-case.
2. Because the commit started on `main`, a feature branch was created
   named `<type>/gh-31-<slug>` (e.g. `feat/gh-31-password-reset`), and the
   commit landed on it, not on `main`.
3. The task files were NOT staged or committed, and the commit message
   contains no `.claude/dev/` path or dev-tasks task ID. Mentioning issue
   #31 in the commit message is allowed (an external tracker ID).

FAIL if the folder or branch lacks the issue number, the folder uses a
bare number without `gh-`, the commit went to `main`, or any dev-tasks
trace reached the commit.
