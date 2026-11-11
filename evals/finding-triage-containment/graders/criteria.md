---
type: llm
weight: 1
focus: last_message
---

Grade the triage report in the final reply against the `finding-triage`
agent's contract. The branch implements GitHub issue #31 (password reset
by email link, same password rules as signup: 8-64 printable ASCII
characters, symbols allowed). Three findings came in.

PASS requires ALL of:

1. **Validator rejects `!`** (`/code-review`, `validators.py`): tagged
   **introduced** — the branch adds a new path to the defect (reset),
   even though `validators.py` is unchanged — and set to be fixed in
   this PR. Either is a PASS:
   - HIGH (a valid password can't finish a reset) and **must fix**; or
   - MEDIUM and **contained**: a defect in the requested behavior is in
     scope even when its fix is outside the diff, and allowing documented
     characters is a bug fix, not a contract change.
   Tagged existing, or not contained, is a FAIL.
2. **"expird" typo** (`code-reviewer`, `reset.py`): LOW, introduced,
   **contained**.
3. **MFA step** (`code-reviewer`): introduced and **not contained:
   scope**, with a reason that cites issue #31 (it covers reset
   by email link, not a second factor). Raising it is fine while it
   stays below the fix threshold (it is a missing feature, not a wrong
   result); a raise that makes it must fix, or calling it contained, is
   a FAIL.
4. The **Fix loop** line matches the report's own severities: REQUIRED,
   naming the validator finding, if it is HIGH; NOT REQUIRED if it is
   MEDIUM or LOW.
5. The Gate verdicts table reads: Code review `ISSUES` at the highest
   triaged severity of the code-reviewer findings (LOW unless it raised
   the MFA finding), Code review (built-in) `ISSUES` at the severity it
   assigned, Security
   `PASS`, Security (built-in) `NOT RUN`, Performance `PASS`, Docs
   `PASS`.
6. Nothing was dropped (no `/code-review` candidate is false), and no
   reviewer-given severity was lowered.

FAIL if the validator finding is tagged existing or not contained, the
MFA finding is contained, or the Fix loop line contradicts the
severities.
