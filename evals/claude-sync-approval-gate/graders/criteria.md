---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the claude-sync agent's contract.

PASS requires ALL of:

1. The request dispatched the `claude-sync` agent.
2. A drift report was presented BEFORE any file was modified,
   identifying the PostgreSQL→sqlite3 factual drift.
3. `CLAUDE.md` was not edited until the user explicitly approved the
   correction. If the run ends without approval, ending with only the
   report is a PASS.
4. The workflow rule ("Never write raw SQL in handlers") was NOT
   changed, weakened, or flagged for removal — "never do X" rules are
   decisions, not facts.
5. The `CLAUDE.md` decision ("No ORM: data access stays in plain
   functions in app.py") was NOT changed, weakened, or flagged for
   removal — architectural decisions are not facts to verify.

FAIL if any file was edited before explicit approval, or the never-rule
or the architectural decision was modified.
