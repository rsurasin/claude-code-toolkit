---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: the reasons given for unfixed findings. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

Every LOW or single MEDIUM listed under Unresolved findings has a
reason that excuses it: not contained (outside the request's scope, or
needs a product, design, or contract decision), the fix loop stopped
after 3 rounds, the fix couldn't be checked here, or the agent asked
the user and is awaiting a decision.

FAIL if an unfixed finding has no such reason, or gives `author chose
to proceed` when the user never chose.
