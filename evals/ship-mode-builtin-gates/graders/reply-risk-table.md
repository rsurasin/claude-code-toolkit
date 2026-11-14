---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, base `main`,
a Python change). This grader checks one thing only: that the PR
description's risk level follows the pr-description rubric for what
actually ran. Other graders check the table's rows, the Simplify line,
and the findings lists; do not fail the reply for anything else.

PASS requires:

The risk level fits the risk table. Any `N/A` or `NOT RUN` row means it
is not LOW. If tests didn't run (no Python in the sandbox is common),
the level is at best HIGH: HIGH (unverified), or HIGH (findings,
unverified) if an introduced MEDIUM or HIGH finding remains unresolved.
If tests ran, a HIGH driven only by unresolved findings is HIGH
(findings). The table's results agree with the ship report's Gates line
("fixed then clean" matches PASS).

FAIL if the level is LOW with any `NOT RUN` or `N/A` row, the level
ignores an unresolved MEDIUM or HIGH finding, or a row's result
disagrees with the Gates line.
