---
max_turns: 40
timeout_seconds: 900
allowed_tools: [Read, Glob, Grep, Agent, Bash, Write, Edit]
---

Set up a scratch project whose branch introduces a race, then have its
gate findings triaged. The eval sandbox keeps placeholder entries such as
`.claude/` and `.bashrc` in this directory: leave them alone, and stage
files by name in every commit (`git add <file>`), since `git add -A`
trips on them.

1. In the current working directory, run `git init` and create
   an initial commit on `main` with `accounts.py`:

   ```python
   import sqlite3


   class InsufficientFunds(Exception):
       pass


   def transfer(db: sqlite3.Connection, src: int, dst: int, amount: int) -> None:
       if amount <= 0:
           raise ValueError("amount must be positive")
       with db:
           cur = db.execute(
               "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
               (amount, src, amount),
           )
           if cur.rowcount != 1:
               raise InsufficientFunds(src)
           db.execute("UPDATE accounts SET balance = balance + ? WHERE id = ?", (amount, dst))
   ```
2. On a `feat/gh-40-transfer-fee` branch, commit a version that adds
   `FEE = 1` as the file's last line (so the lines above keep their
   numbers) and replaces the guarded debit inside `with db:` with:

   ```python
           (balance,) = db.execute("SELECT balance FROM accounts WHERE id = ?", (src,)).fetchone()
           if balance < amount + FEE:
               raise InsufficientFunds(src)
           db.execute("UPDATE accounts SET balance = ? WHERE id = ?", (balance - amount - FEE, src))
   ```

   keeping the `amount <= 0` check and the credit to `dst` as they are.
3. Then dispatch the **`finding-triage` agent** to triage this branch's
   gate findings, and reply with its report verbatim. Give it exactly
   this input:

   - Range: `main...HEAD` (base `main`).
   - Scope: GitHub issue #40, "Charge a flat 1-unit fee on transfers".
     Body: "Each transfer debits the sender amount + 1. Concurrent
     transfers must never overdraw an account."
   - Gate reports:
     - `code-reviewer`: **Verdict:** ISSUES (LOW).
       `- **[LOW]** [introduced] accounts.py:12 — the debit now reads the balance and writes back a value computed in Python; consider restoring the single UPDATE. Fix: fold the fee into the guarded UPDATE.`
     - `/code-review`: two findings, no severity:
       `accounts.py:12-15 — read-then-write race: two concurrent transfers from the same account can both read balance 10, both pass the check, and both write 10 - amount - FEE, so one debit is lost and the account can end above its true balance or overdraw. The removed UPDATE ... AND balance >= ? made the check and the debit atomic.`
       `accounts.py:9 — transfer accepts amount = 0 and charges the fee for moving nothing.`
     - `security-auditor`: **Verdict:** PASS, no findings.
     - `perf-audit`: no findings.
     - `doc-sync`: no findings.
     - `/security-review`: NOT RUN (no `origin` remote).
