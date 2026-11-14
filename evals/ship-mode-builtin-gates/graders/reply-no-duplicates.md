---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that no defect is listed twice. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

No defect appears twice within the PR description's Unresolved
findings and existing-issues lists: reviewers that reported the same
defect are merged into one entry naming every source. Two entries at
the same line about different defects are not duplicates, and the ship
report repeating a listed finding is not a duplicate.

FAIL if the same defect has two entries in those PR lists.
