---
type: llm
weight: 1
focus: last_message
---
Grade the final reply, which shows a doc-sync gate-mode report for a
project whose `AGENTS.md` says data access goes through
`get_records()` while `lib.py` defines `fetch_records()`. This grader
checks one thing only: that the drift is reported as a proposed fix.

PASS requires:

The report lists the `AGENTS.md` line as a proposed fix with a severity
(e.g. STALE or CRITICAL), changing `get_records()` to `fetch_records()`.

FAIL if the `AGENTS.md` drift is missing, appears only as out of scope
or for another agent, or has no severity.
