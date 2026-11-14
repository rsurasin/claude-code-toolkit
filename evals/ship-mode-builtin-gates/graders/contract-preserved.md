---
type: llm
weight: 1
focus: { source: file, path: accounts.py }
---

Grade this `accounts.py`, the code as ship mode left it on the
`feature/min-balance` branch. The branch was asked to make a transfer
leave at least `MIN_BALANCE` (10) in the source account; ship mode then
fixed review findings. This grader checks that those fixes kept the
public contract `main` had. On `main`, `transfer(db, src, dst, amount)`
returned nothing, raised `ValueError` for a non-int, bool, or
non-positive `amount` or for `src == dst`, raised `InsufficientFunds`
when `src` was missing or its balance was less than `amount`, raised
`AccountNotFound` when `dst` was missing, and accepted any sqlite3
connection, in any transaction state, for `db`. Those two classes were
its only exception classes.

PASS requires ALL of:

1. `transfer` keeps the signature `transfer(db, src, dst, amount)`: no
   parameter added, removed, renamed, or reordered, and it still returns
   nothing.
2. No new exception class: the file defines only `InsufficientFunds` and
   `AccountNotFound` (plus any helper that is not an exception).
3. No new precondition rejects a call `main` accepted: no check that
   raises for the connection's mode or transaction state (e.g. testing
   `isolation_level`, `autocommit`, or `in_transaction`, or an explicit
   `BEGIN` that fails when the caller already has a transaction open),
   for the type or value of `src` or `dst`, or for an `amount` `main`
   accepted.

The minimum-balance check itself is the requested behavior, not a
violation: raising `InsufficientFunds` when the transfer would leave
`src` below `MIN_BALANCE` is correct, as is a constant like
`MIN_BALANCE`. Clearer error messages, docstring changes, and comments
are fine.

FAIL if the file is missing or any item above fails.
