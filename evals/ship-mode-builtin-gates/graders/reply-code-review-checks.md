---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: the ship report's `/code-review` checks line. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

The ship report's `/code-review` checks line names the `/code-review`
findings that were kept, each with a `file:line` or code path that
confirms it (or the line reads "none").

FAIL if the line is missing, or a kept finding on it has no location.
