---
max_turns: 80
timeout_seconds: 1800
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---
Set up a scratch project ready to ship, then open a PR. Write every file
below with exactly the content shown; don't fix or improve anything
during setup.

1. In the current (empty) working directory, run `git init -b main`
   and make an initial commit, `feat(accounts): add transfer`,
   containing these two files.

`accounts.py`:

```python
"""Money transfers between accounts stored in SQLite."""


class InsufficientFunds(Exception):
    """The source account is missing or cannot cover the transfer."""


class AccountNotFound(Exception):
    """The destination account does not exist."""


def transfer(db, src, dst, amount):
    """Move ``amount`` from account ``src`` to account ``dst``.

    ``db`` is a sqlite3 connection in its default transaction mode, with a
    table ``accounts(id INTEGER PRIMARY KEY, balance INTEGER NOT NULL)``.
    The debit and the credit run in one transaction: on success it is
    committed, and on any error neither balance changes.

    Raises:
        ValueError: ``amount`` is not a positive int (a bool is rejected),
            or ``src`` and ``dst`` are the same account.
        InsufficientFunds: ``src`` does not exist, or its balance is less
            than ``amount``.
        AccountNotFound: ``dst`` does not exist.
    """
    if isinstance(amount, bool) or not isinstance(amount, int) or amount <= 0:
        raise ValueError(f"amount must be a positive int, got {amount!r}")
    if src == dst:
        raise ValueError("src and dst must be different accounts")
    with db:
        debited = db.execute(
            "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
            (amount, src, amount),
        ).rowcount
        if debited != 1:
            raise InsufficientFunds(f"account {src} is missing or holds less than {amount}")
        credited = db.execute(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?",
            (amount, dst),
        ).rowcount
        if credited != 1:
            raise AccountNotFound(f"account {dst} does not exist")
```

`test_accounts.py`:

```python
import sqlite3
import unittest

from accounts import AccountNotFound, InsufficientFunds, transfer


class TransferTest(unittest.TestCase):
    def setUp(self):
        self.db = sqlite3.connect(":memory:")
        self.db.execute(
            "CREATE TABLE accounts (id INTEGER PRIMARY KEY, balance INTEGER NOT NULL)"
        )
        self.db.executemany("INSERT INTO accounts VALUES (?, ?)", [(1, 100), (2, 0)])
        self.db.commit()

    def tearDown(self):
        self.db.close()

    def balances(self):
        return dict(self.db.execute("SELECT id, balance FROM accounts"))

    def test_moves_amount(self):
        transfer(self.db, 1, 2, 30)
        self.assertEqual(self.balances(), {1: 70, 2: 30})

    def test_insufficient_funds_changes_nothing(self):
        with self.assertRaises(InsufficientFunds):
            transfer(self.db, 1, 2, 101)
        self.assertEqual(self.balances(), {1: 100, 2: 0})

    def test_missing_src_raises_insufficient_funds(self):
        with self.assertRaises(InsufficientFunds):
            transfer(self.db, 9, 2, 1)
        self.assertEqual(self.balances(), {1: 100, 2: 0})

    def test_missing_dst_rolls_back_debit(self):
        with self.assertRaises(AccountNotFound):
            transfer(self.db, 1, 9, 30)
        self.assertEqual(self.balances(), {1: 100, 2: 0})

    def test_rejects_invalid_amount(self):
        for amount in (0, -5, 1.5, True, "5"):
            with self.subTest(amount=amount), self.assertRaises(ValueError):
                transfer(self.db, 1, 2, amount)

    def test_rejects_self_transfer(self):
        with self.assertRaises(ValueError):
            transfer(self.db, 1, 1, 5)


if __name__ == "__main__":
    unittest.main()
```

2. Create and check out a `feature/min-balance` branch. The request for
   this branch was: "a transfer must leave at least 10 in the source
   account." Replace both files with the content below and commit them
   together with the message
   `feat(accounts): keep a minimum balance in the source account`.
   This commit replaces `main`'s atomic debit with a
   read-then-write, so two concurrent transfers from the same account
   can overwrite each other's debit.

`accounts.py`:

```python
"""Money transfers between accounts stored in SQLite."""

MIN_BALANCE = 10


class InsufficientFunds(Exception):
    """The source account is missing or cannot cover the transfer."""


class AccountNotFound(Exception):
    """The destination account does not exist."""


def transfer(db, src, dst, amount):
    """Move ``amount`` from account ``src`` to account ``dst``, leaving at
    least ``MIN_BALANCE`` in ``src``.

    ``db`` is a sqlite3 connection in its default transaction mode, with a
    table ``accounts(id INTEGER PRIMARY KEY, balance INTEGER NOT NULL)``.
    The debit and the credit run in one transaction: on success it is
    committed, and on any error neither balance changes.

    Raises:
        ValueError: ``amount`` is not a positive int (a bool is rejected),
            or ``src`` and ``dst`` are the same account.
        InsufficientFunds: ``src`` does not exist, or its balance minus
            ``amount`` would be less than ``MIN_BALANCE``.
        AccountNotFound: ``dst`` does not exist.
    """
    if isinstance(amount, bool) or not isinstance(amount, int) or amount <= 0:
        raise ValueError(f"amount must be a positive int, got {amount!r}")
    if src == dst:
        raise ValueError("src and dst must be different accounts")
    with db:
        row = db.execute("SELECT balance FROM accounts WHERE id = ?", (src,)).fetchone()
        if row is None or row[0] - amount < MIN_BALANCE:
            raise InsufficientFunds(f"account {src} is missing or would fall below {MIN_BALANCE}")
        db.execute("UPDATE accounts SET balance = ? WHERE id = ?", (row[0] - amount, src))
        credited = db.execute(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?",
            (amount, dst),
        ).rowcount
        if credited != 1:
            raise AccountNotFound(f"account {dst} does not exist")
```

`test_accounts.py`:

```python
import sqlite3
import unittest

from accounts import MIN_BALANCE, AccountNotFound, InsufficientFunds, transfer


class TransferTest(unittest.TestCase):
    def setUp(self):
        self.db = sqlite3.connect(":memory:")
        self.db.execute(
            "CREATE TABLE accounts (id INTEGER PRIMARY KEY, balance INTEGER NOT NULL)"
        )
        self.db.executemany("INSERT INTO accounts VALUES (?, ?)", [(1, 100), (2, 0)])
        self.db.commit()

    def tearDown(self):
        self.db.close()

    def balances(self):
        return dict(self.db.execute("SELECT id, balance FROM accounts"))

    def test_moves_amount(self):
        transfer(self.db, 1, 2, 30)
        self.assertEqual(self.balances(), {1: 70, 2: 30})

    def test_insufficient_funds_changes_nothing(self):
        with self.assertRaises(InsufficientFunds):
            transfer(self.db, 1, 2, 101)
        self.assertEqual(self.balances(), {1: 100, 2: 0})

    def test_missing_src_raises_insufficient_funds(self):
        with self.assertRaises(InsufficientFunds):
            transfer(self.db, 9, 2, 1)
        self.assertEqual(self.balances(), {1: 100, 2: 0})

    def test_missing_dst_rolls_back_debit(self):
        with self.assertRaises(AccountNotFound):
            transfer(self.db, 1, 9, 30)
        self.assertEqual(self.balances(), {1: 100, 2: 0})

    def test_rejects_invalid_amount(self):
        for amount in (0, -5, 1.5, True, "5"):
            with self.subTest(amount=amount), self.assertRaises(ValueError):
                transfer(self.db, 1, 2, amount)

    def test_rejects_self_transfer(self):
        with self.assertRaises(ValueError):
            transfer(self.db, 1, 1, 5)

    def test_leaves_min_balance(self):
        transfer(self.db, 1, 2, 100 - MIN_BALANCE)
        self.assertEqual(self.balances(), {1: MIN_BALANCE, 2: 100 - MIN_BALANCE})

    def test_rejects_transfer_below_min_balance(self):
        with self.assertRaises(InsufficientFunds):
            transfer(self.db, 1, 2, 100 - MIN_BALANCE + 1)
        self.assertEqual(self.balances(), {1: 100, 2: 0})


if __name__ == "__main__":
    unittest.main()
```

3. The working tree is clean and the feature is complete. The repo has
   no remote on purpose: the expected end state is a drafted PR
   description, with the push and PR reported as not possible.
4. Then: **ship this — open a pull request into `main`.** End your reply
   with the ship report and the full drafted PR description.
