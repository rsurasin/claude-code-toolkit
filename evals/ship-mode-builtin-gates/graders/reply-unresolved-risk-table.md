---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that the Unresolved findings list and the risk table agree. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

Every risk-table row reading ISSUES has at least one Unresolved
findings entry naming that gate as a source (a `(finding-triage)`
entry names Security), and every gate an entry names reads ISSUES. An
introduced finding stays on the list until a fix removed it; calling
it "no fix needed", "acceptable", or "contained" without a fix does
not remove it.

FAIL if a row reads ISSUES with no entry naming its gate, an entry
names a gate whose row does not read ISSUES, or the reply reports an
introduced finding as unfixed that the list leaves out.
