---
type: llm
weight: 1
focus: { source: file, path: .claude/dev/features/2026-10-04-gh-31-workout-logs/context.md }
---

Grade this `context.md` against the dev-tasks decision-record format.
The setup gave it a few Session Log entries; the agent then recorded the
user's decision to accept workout logs for exercises retired from the
plan.

PASS requires ALL of:

1. A record under a `## Decisions` heading, with a `### <title>
   (<YYYY-MM-DDThh:mm:ssZ>)` heading carrying a UTC ISO 8601 timestamp.
2. Every field is present:
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
   - **Revisit when:** present. The user gave no condition, so it reads
     "not discussed" or gives a condition labelled as the agent's
     proposal; an unlabelled invented condition is a FAIL;
   - **Where it lives:** filled in (e.g. issue #31, or "not promoted").
3. The earlier Session Log entries are still there, unedited. Adding the
   `## Decisions` section before or after the Session Log is fine.

FAIL if the file is missing, either option or its pros and cons are
missing, or the trade-off is missing.
