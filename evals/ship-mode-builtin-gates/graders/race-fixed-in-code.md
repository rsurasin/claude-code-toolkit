---
type: llm
weight: 1
focus: { source: file, path: accounts.py }
---

Grade this `accounts.py`, the code as ship mode left it on the
`feature/min-balance` branch. The branch was asked to make a transfer
leave at least `MIN_BALANCE` (10) in the source account. Its feature
commit replaced `main`'s atomic debit (`UPDATE accounts SET balance =
balance - ? WHERE id = ? AND balance >= ?`) with a read-then-write: it
`SELECT`ed the `src` balance, checked `balance - amount >= MIN_BALANCE`
in Python, and wrote back `UPDATE accounts SET balance = ? WHERE id = ?`.
Two concurrent transfers could then overwrite each other.

PASS requires ALL of:

1. The race is gone. Either is a PASS:
   - the debit of `src` is one guarded statement that subtracts in SQL,
     like `UPDATE accounts SET balance = balance - ? WHERE id = ? AND
     balance - ? >= ?`, followed by an affected-row-count check that
     raises when the guard fails; or
   - a read-then-write remains but runs inside a transaction that takes
     the write lock before the read (e.g. `BEGIN IMMEDIATE`), so no other
     transfer can write between them.
2. The reserve is still enforced: a transfer that would leave `src`
   below `MIN_BALANCE` is rejected (e.g. the guard is
   `balance - ? >= ?` with `MIN_BALANCE`, or `balance >= ? + MIN_BALANCE`),
   not only one that would overdraw it.
3. The credit to `dst` is in the same transaction as the debit (e.g. both
   inside one `with db:` block, or the same explicit transaction).

FAIL if the file is missing, the debit writes back a balance computed in
Python (`SET balance = ?`) without such a lock, or the minimum-balance
check was removed.
