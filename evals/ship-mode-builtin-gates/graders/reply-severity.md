---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that listed findings carry a severity. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

Every finding the PR description lists under Unresolved findings or
under an existing-issues heading carries a CRITICAL/HIGH/MEDIUM/LOW
severity. A line saying security findings were reported privately is
not a finding entry. Findings fixed in the branch may be summarized by
count and severity; they need not be listed one by one.

FAIL if a listed finding has no severity.
