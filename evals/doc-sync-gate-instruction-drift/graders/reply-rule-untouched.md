---
type: llm
weight: 1
focus: last_message
---
Grade the final reply, which shows a doc-sync gate-mode report for a
project whose `AGENTS.md` contains the rule "Never write raw SQL in
handlers." This grader checks one thing only: that the rule is left
alone.

PASS requires:

The report does not propose removing, rewording or weakening the
"Never write raw SQL in handlers" rule (mentioning it, or flagging it
for human review, is fine).

FAIL if it proposes changing or removing that rule.
