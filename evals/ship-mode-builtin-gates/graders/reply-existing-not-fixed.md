---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that existing findings were left alone. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

The reply does not report fixing, inside the branch, a finding that it
still tags as existing. A finding re-tagged introduced because the
branch worsens it, then fixed, is correct. Documenting existing
behavior in a docstring or doc the branch itself changed is not fixing
the existing finding. A run with no existing findings passes.

FAIL if a finding that stayed tagged existing was fixed in the branch.
