---
type: llm
weight: 1
focus: last_message
---

Grade the triage report in the final reply against the `finding-triage`
agent's contract. The branch replaced an atomic debit with a
read-then-write. `code-reviewer` reported it as LOW; `/code-review`
reported it with a failure scenario and no severity, plus a second
candidate about `amount = 0`.

PASS requires ALL of:

1. **The race is one merged finding at HIGH or CRITICAL**, naming both
   sources (`code-reviewer` and `/code-review`), tagged introduced, and
   marked **must fix**. `code-reviewer`'s LOW was raised, not kept:
   a reachable race that leaves inconsistent balances is HIGH however
   small the fix. Two separate entries for the race, or the race at
   LOW or MEDIUM, is a FAIL.
2. **Fix loop: REQUIRED**, naming the race.
3. **The `amount = 0` candidate is dropped** under Dropped, with the
   reason that `transfer` raises `ValueError` for `amount <= 0` (the
   failure scenario can't happen). Keeping it as a finding is a FAIL.
4. The Triage changes section records raising `code-reviewer`'s LOW and
   merging the two race reports.
5. The Gate verdicts table reads: Code review `ISSUES` at code-reviewer's
   own severity (LOW) or higher, Code review (built-in) `ISSUES (HIGH)`
   or `ISSUES (CRITICAL)`, Security `PASS`, Security (built-in)
   `NOT RUN`, Performance `PASS`, Docs `PASS`.

FAIL if the race is rated below HIGH, the fix loop is NOT REQUIRED, or
the `amount = 0` candidate survives.
