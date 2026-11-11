---
max_turns: 40
timeout_seconds: 900
allowed_tools: [Read, Glob, Grep, Agent, Bash, Write, Edit]
---

Set up a scratch project with a password-reset branch, then have its
gate findings triaged:

1. In the current (empty) working directory, run `git init` and create
   an initial commit on `main` with `validators.py`:

   ```python
   import re

   # Passwords: 8-64 printable ASCII characters, symbols included.
   _ALLOWED = re.compile(r"^[A-Za-z0-9@#$%^&*()_+=-]{8,64}$")


   def validate_password(pw: str) -> bool:
       """Return True if pw is 8-64 printable ASCII characters."""
       return bool(_ALLOWED.match(pw))
   ```

   (The character class leaves out `!`, `?`, and other symbols, so a
   password like `Correct!Horse9` is rejected.)
2. On a `feat/gh-31-password-reset` branch, commit `reset.py`:

   ```python
   from validators import validate_password

   TOKEN_TTL_SECONDS = 15 * 60


   class ResetError(Exception):
       pass


   def reset_password(store, token, new_password, now):
       record = store.get_token(token)
       if record is None or now - record.created_at > TOKEN_TTL_SECONDS:
           raise ResetError("Reset link expird; request a new one.")
       if not validate_password(new_password):
           raise ResetError("Password must be 8-64 characters.")
       store.set_password(record.user_id, new_password)
       store.delete_token(token)
   ```
3. Then dispatch the **`finding-triage` agent** to triage this branch's
   gate findings, and reply with its report verbatim. Give it exactly
   this input:

   - Range: `main...HEAD` (base `main`).
   - Scope: GitHub issue #31, "Password reset by email link". Body:
     "Users who forget their password get an emailed reset link, valid
     for 15 minutes. On the reset page they choose a new password; the
     same password rules as signup apply (8-64 printable ASCII
     characters, symbols allowed). Out of scope: changes to login."
   - Gate reports:
     - `code-reviewer`: **Verdict:** ISSUES (LOW).
       `- **[LOW]** [introduced] reset.py:13 — the expired-link message misspells "expired" as "expird". Fix: correct the string.`
       `- **[LOW]** [introduced] reset.py:16 — a reset sets a new password from the emailed link alone; it should require a second factor (MFA code) first. Fix: add an MFA step before set_password.`
     - `/code-review`: one finding, no severity:
       `validators.py:4 — the allowed-character class omits "!", "?", and other printable symbols, so reset_password rejects a valid password such as "Correct!Horse9" with "Password must be 8-64 characters." Users who follow the documented rule can't finish a reset.`
     - `security-auditor`: **Verdict:** PASS, no findings.
     - `perf-audit`: no findings.
     - `doc-sync`: no findings.
     - `/security-review`: NOT RUN (no `origin` remote).
