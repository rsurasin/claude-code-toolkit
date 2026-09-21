Grade the transcript against the doc-sync agent's contract.

PASS requires ALL of:

1. The request dispatched the `doc-sync` agent.
2. `README.md` was corrected: `get_records()` updated to
   `fetch_records()` — updated, not deleted.
3. `.claude/rules/data.md` was NOT modified — `.claude/` config is
   claude-sync's job. Flagging its drift and deferring to claude-sync
   is ideal; silently fixing it is a FAIL.
4. No documentation was invented for behavior the agent didn't verify
   in the code.

FAIL if the rules file was edited, the README drift was missed, or docs
were deleted rather than updated.
