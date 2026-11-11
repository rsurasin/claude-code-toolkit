---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the dev-tasks skill's decision-record rule:
when the decision was recorded, and what happened to the code. Separate
graders read the written `context.md` and `plan.md` to check the record
itself.

Note: the eval sandbox may refuse writes under `.claude/`, in the setup or
later. If it did, that is the environment, not a failure — grade the
writes the agent attempted (the commands in the trace). Writing the files
with a shell heredoc instead of the Write tool is fine.

Note: the trace keeps only the first and last messages and elides the
middle (marked "[…N messages elided…]"). If the decision-record write
falls inside the elided part, don't fail item 1 for not seeing it: judge
it from what is visible — a visible write of the record after the
`logs.py` edit is a FAIL; a code edit with the record write hidden before
it is not.

PASS requires ALL of:

1. The decision record was written to the task folder's `context.md`
   before the agent edited `logs.py`, in the same turn. Recording it only
   at the end, only in chat, or not at all is a FAIL.
2. `accept_log(log)` was added to `logs.py`. No `.claude/dev/` file was
   staged or committed, and no commit message references the record.

FAIL if the record was visibly written after the code change, or only
reported in chat.
