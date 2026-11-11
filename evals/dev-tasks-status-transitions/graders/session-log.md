---
type: llm
weight: 1
focus: { source: file, path: .claude/dev/features/2026-10-04-gh-31-password-reset/context.md }
---

Grade this `context.md` against the dev-tasks Session Log rule. The setup
created it; the agent then recorded that PR #41 merged into `dev` as
`9f3c2e1`.

PASS requires ALL of:

1. A Session Log entry records the status change to merged to dev
   (`9f3c2e1`), with a UTC ISO 8601 timestamp
   (`YYYY-MM-DDThh:mm:ssZ`) and the `status: merged to dev (9f3c2e1), not
   released` text. Extra context on the same line (e.g. that the merge
   couldn't be checked on GitHub) is fine.
2. Nothing visibly lost: the setup's own header lines are still above
   the log. You can't see the setup's version, so fail this only if the
   file plainly starts over (e.g. holds the new entry alone). The setup
   may have written no Session Log; the agent adding the heading is fine.

FAIL if the transition isn't logged, the timestamp is missing or not
UTC, or earlier content was removed or rewritten.
