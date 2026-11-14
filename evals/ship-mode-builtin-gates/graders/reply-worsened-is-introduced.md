---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that a finding the branch worsens is not left tagged existing. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

The reply does not keep a finding tagged `existing` while saying the
branch makes it worse. Re-tagged introduced and fixed, or listed under
Unresolved findings, is correct. A run with no such finding passes.

FAIL if a finding is tagged existing and the reply says the branch
worsens it.
