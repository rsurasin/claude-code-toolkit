Grade the transcript against the dev-tasks skill's decision-record rule.

PASS requires ALL of:

1. A decision record was appended to the task folder's `context.md`
   under `## Decisions` BEFORE the agent edited `logs.py`, in the same
   turn. Recording it only at the end, only in chat, or not at all is a
   FAIL.
2. The record follows the documented format, with every field:
   - **Decision:** accept logs for retired exercises, keeping the retired
     name (in product terms);
   - **Context:** the app can send logs for exercises retired from the
     plan;
   - **Options considered:** BOTH options (reject after a 7-day grace
     period; accept and keep the name), each with its own pros AND cons;
   - **Chosen and why:** accept, because losing a user's workout is worse
     than a messy report;
   - **Trade-offs accepted:** reports can show exercises no longer in
     the plan;
   - **Revisit when:** a concrete condition (e.g. reports become
     unusable, or retired exercises must be blocked for safety);
   - **Where it lives:** filled in (e.g. the branch's issue #31, or "not
     promoted").
3. The heading carries a UTC ISO 8601 timestamp obtained via `date -u`.
4. Existing `context.md` content is unchanged (append-only).
5. `plan.md` → Key Decisions links to the record by title/timestamp
   rather than holding a one-line rationale.
6. `accept_log(log)` was added to `logs.py`. No `.claude/dev/` file was
   staged or committed, and no commit message references the record.

FAIL if either option or its pros and cons are missing, the trade-off is
missing, the record was written after the code change, or existing
`context.md` entries were rewritten.
