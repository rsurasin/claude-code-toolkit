---
type: llm
weight: 1
---

Grade the transcript against the dev-tasks skill's status and release
tracking contract.

PASS requires ALL of:

1. The password-reset folder's `tasks.md` now ends with exactly
   `Status: merged to dev (9f3c2e1), not released`: the fixed wording,
   the base branch `dev`, the reported SHA, on the last line. Marking it
   `released`, or inventing other wording ("merged", "done",
   "awaiting release"), is a FAIL.
2. The transition was appended to that folder's `context.md` Session Log
   with a UTC ISO 8601 timestamp; existing `context.md` content is
   unchanged.
3. The catch-up lists both folders still waiting on a release
   (password-reset and cap-sets) as `merged to dev ..., not released`,
   and mentions the cap-sets folder's open release step (migration
   `0042_cap_sets`).
4. The streak-badges folder's status is untouched (`Status: in progress`)
   and the cap-sets folder is NOT marked released.
5. No `.claude/dev/` file was staged or committed.

FAIL if any status line uses wording outside the four fixed states, a
folder is marked `released` without a merged release PR, or the pending
release folders are not surfaced.
