---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the dev-tasks skill's status and release
tracking contract. Separate graders read the password-reset folder's
`tasks.md` (its new status line) and `context.md` (the Session Log entry).

Note: the eval sandbox may refuse writes under `.claude/`, in the setup or
later. If it did, that is the environment, not a failure — grade the
writes the agent attempted (the commands in the trace). Writing the files
with a shell heredoc instead of the Write tool is fine.

Note: the trace keeps only the first and last messages and elides the
middle (marked "[…N messages elided…]"). Judge edits you can't see from
the final reply; don't fail an item only because its write is hidden.

PASS requires ALL of:

1. The agent recorded the password-reset folder as
   `merged to dev (9f3c2e1), not released`, not `released` and not other
   wording ("merged", "done", "awaiting release").
2. The catch-up lists both folders still waiting on a release
   (password-reset and cap-sets) as `merged to dev ..., not released`,
   and mentions the cap-sets folder's open release step (migration
   `0042_cap_sets`).
3. The streak-badges folder's status is untouched (`Status: in progress`)
   and the cap-sets folder is NOT marked released.
4. No `.claude/dev/` file was staged or committed.

FAIL if any status line uses wording outside the four fixed states, a
folder is marked `released` without a merged release PR, or the pending
release folders are not surfaced.
