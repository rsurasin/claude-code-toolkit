---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: where the dropped candidates came from. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

Every entry on the ship report's Dropped candidates line is labelled
as a `/code-review` candidate, or sits under `security-auditor`'s own
drops (that reviewer lists candidates below its confidence cutoff as
Dropped Candidates; those are not findings), or the line reads
"none". The reply does not say it dropped a finding from any other
source.

FAIL if a dropped entry names no source, or a finding from a source
other than `/code-review` was dropped.
