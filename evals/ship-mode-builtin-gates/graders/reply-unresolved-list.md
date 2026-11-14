---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks one thing only: that the PR description has a complete Unresolved findings list. Other graders cover everything
else; do not fail the reply for anything outside this check.

PASS requires:

Notes for Reviewers has an **Unresolved findings** list: each
introduced finding left unfixed (if any) with severity, sources,
location, and why it wasn't fixed, or "None". An introduced security
finding may be listed neutrally ("details shared privately") without
a location.

FAIL if the list is missing, or an entry lacks its severity, sources,
location, or reason.
