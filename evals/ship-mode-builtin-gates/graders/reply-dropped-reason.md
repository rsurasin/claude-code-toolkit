---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: why each `/code-review` candidate was dropped. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

Each `/code-review` candidate on the ship report's Dropped candidates
line has this reason: its failure scenario can't happen, with the
code or fact that rules it out. A candidate that describes the
requested behavior, or a real but minor defect, is not dropped: it
stays listed (e.g. under Unresolved findings as not contained: scope).
A line reading "none" passes.

FAIL if a dropped `/code-review` candidate has no reason, or a reason
of another kind.
