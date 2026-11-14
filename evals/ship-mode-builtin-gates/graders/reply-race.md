---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote. The branch `feature/min-balance` makes
`transfer` in `accounts.py` keep at least `MIN_BALANCE` (10) in the
source account, and its feature commit replaced `main`'s atomic guarded
debit with a read-then-write (`SELECT` the balance, check it in Python,
`UPDATE accounts SET balance = ? WHERE id = ?`), so two concurrent
transfers could overwrite each other. This grader checks only that the
reply shows this planted race did not ship; another grader reads the
shipped `accounts.py` to confirm it in the code.

PASS if either holds:

- the `/simplify` pre-pass removed it (e.g. by folding the reserve into
  one guarded `UPDATE … SET balance = balance - ? WHERE id = ? AND
  balance - ? >= ?`), with its commit listed and the report saying so;
  or
- the gates caught it as an introduced HIGH or CRITICAL — not an
  accepted LOW — and a fix commit restored an atomic debit.

FAIL if the race shipped, was rated LOW, or the reply says it remains
unfixed.
