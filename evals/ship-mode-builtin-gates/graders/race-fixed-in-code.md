---
type: llm
weight: 1
focus: { source: file, path: accounts.py }
---

Grade this `accounts.py`, the code as ship mode left it on the
`feature/fees` branch. The branch's feature commit replaced an atomic
debit with a read-then-write: it `SELECT`ed the `src` balance, computed
the new balance in Python, and wrote it back with
`UPDATE accounts SET balance = ? WHERE id = ?`. Two concurrent transfers
could then overwrite each other.

PASS requires ALL of:

1. The race is gone. Either is a PASS:
   - the debit of `src` is one guarded statement that subtracts in SQL,
     like `UPDATE accounts SET balance = balance - ? WHERE id = ? AND
     balance >= ?`, followed by an affected-row-count check that raises
     when the guard fails; or
   - a read-then-write remains but runs inside a transaction that takes
     the write lock before the read (e.g. `BEGIN IMMEDIATE`), so no other
     transfer can write between them.
2. The fee is still charged: the amount debited from `src` includes the
   fee (e.g. `amount + fee`), not `amount` alone.
3. The credit to `dst` is in the same transaction as the debit (e.g. both
   inside one `with db:` block, or the same explicit transaction).

FAIL if the file is missing, the debit writes back a balance computed in
Python (`SET balance = ?`) without such a lock, or the fee was removed.
