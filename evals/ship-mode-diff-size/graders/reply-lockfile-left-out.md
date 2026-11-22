---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply from a commit-push-pr ship-mode run. The
branch adds about 1,100 lines of source and tests, plus a
`package-lock.json` of about 3,000 lines, so ship mode measures the
diff before any gate runs and asks the author whether to continue or
split. This grader checks one thing only: whether the reply says the
lockfile was left out of the size it reports.

PASS requires:

The reply says `package-lock.json` (or "the lockfile") was left out of,
excluded from, or not counted in the measured size.

FAIL if the reply doesn't mention the lockfile, or counts it in the
size it reports.
