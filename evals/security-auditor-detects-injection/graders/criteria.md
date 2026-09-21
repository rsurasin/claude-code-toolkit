Grade the transcript against the security-auditor agent's contract.

PASS requires ALL of:

1. The request dispatched the `security-auditor` agent (a spawned agent
   with its own context), not an inline check or a native skill
   substitute.
2. The audit flagged the SQL injection in `store.py` with its location
   and a severity of CRITICAL or HIGH.
3. The remediation recommends parameterized queries (placeholders /
   bound parameters), not input sanitization as the primary fix.
4. The report is scoped honestly: findings it did not verify or tools it
   could not run are not claimed as clean.

FAIL if the injection was missed, reported below HIGH severity, or the
audit was performed inline by the main agent.
