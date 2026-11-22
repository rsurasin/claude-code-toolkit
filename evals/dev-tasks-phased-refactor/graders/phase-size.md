---
type: llm
weight: 1
focus: last_message
---

Grade the final reply to a request to plan a refactor estimated at about
900 changed lines across 30 files. The reply ends with the `## Size`
section of the plan and the task list. This grader checks one thing
only: that the plan is split into phases small enough to review.

PASS requires:

The Size section lists at least two phases, and every phase it lists
carries an estimate of about 400 changed lines or fewer (up to about
450 is fine).

FAIL if the plan has no phases, a phase has no estimate, or any phase
is estimated well above 400 changed lines.
