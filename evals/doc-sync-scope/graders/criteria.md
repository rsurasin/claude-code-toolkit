---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the doc-sync agent's contract.

Note: the eval sandbox may refuse writes under `.claude/`. If setup
couldn't create `.claude/rules/data.md`, that is the environment, not a
failure: grade items 1, 2, and 4, and for item 3 that nothing under
`.claude/` was edited.

PASS requires ALL of:

1. The request dispatched the `doc-sync` agent.
2. The README drift was caught and fixed correctly: `get_records()`
   updated to `fetch_records()`, not deleted. The user's request to
   sync the documentation counts as approval for in-scope doc edits, so
   either is a PASS: the README was edited, or the run stopped with the
   exact proposed change (current text → `fetch_records()`) awaiting
   approval.
3. `.claude/rules/data.md` was NOT modified — `.claude/` config is
   claude-sync's job. Flagging its drift and deferring to claude-sync
   is ideal; silently fixing it is a FAIL.
4. No documentation was invented for behavior the agent didn't verify
   in the code.

FAIL if the rules file was edited, the README drift was missed or
proposed wrongly, or docs were deleted rather than updated.
