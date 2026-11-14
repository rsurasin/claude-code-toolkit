---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that listed findings are tagged introduced or existing. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

For every finding the PR description lists, its list or wording shows
whether it is introduced or existing: an entry under Unresolved
findings is introduced; one under an existing-issues heading is
existing.

FAIL if a listed finding sits in neither kind of list and has no
introduced / existing wording.
