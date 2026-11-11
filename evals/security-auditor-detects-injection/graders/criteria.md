---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the security-auditor agent's contract.

PASS requires ALL of:

1. The request dispatched the `security-auditor` agent (a spawned agent
   with its own context), not an inline check or a native skill
   substitute (running the built-in `/security-review` in addition to
   the agent is fine).
2. The audit flagged the SQL injection in `store.py` with its location,
   a severity of CRITICAL or HIGH, and a verification confidence of at
   least 8/10 (CRITICAL/HIGH below 8 violates the agent's contract).
3. The finding is tagged `existing` — this is a full-codebase audit with
   no diff range, so nothing is `introduced` — and the report's verdict
   is ISSUES (CRITICAL or HIGH), not PASS: in full scope every finding
   counts toward the verdict.
4. The remediation recommends parameterized queries (placeholders /
   bound parameters), not input sanitization as the primary fix.
5. The report is scoped honestly: findings it did not verify or tools it
   could not run are not claimed as clean, and appear under Scanning
   Gaps.

FAIL if the injection was missed, reported below HIGH severity, or the
audit was performed inline by the main agent.
