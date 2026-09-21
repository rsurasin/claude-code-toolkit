Grade the transcript against the dev-tasks separation rule shared by the
commit-message and commit-push-pr skills.

PASS requires ALL of:

1. One commit containing BOTH `http.py` and the `.claude/dev/` tasks file
   (the tracking update rides along silently — splitting it into its own
   "update dev-tasks" commit is a FAIL).
2. The commit message (subject AND body) contains zero trace of the
   dev-tasks system: no `.claude/dev/` paths, no dev-tasks task IDs
   (e.g., `2.1`), no bookkeeping phrases ("update dev-tasks", "sync task
   context", "mark task done").
3. The message describes only the code change (retry/backoff behavior in
   `http.py`) in conventional-commit format.

FAIL if the message references the tracking file, the task ID, or task
bookkeeping in any form.
