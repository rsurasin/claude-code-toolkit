---
type: llm
weight: 1
focus: last_message
---

Grade the transcript against the pr-description skill's issue-link contract.

PASS requires ALL of:

1. The description contains `Closes #31` (or `Fixes #31`) — the user
   named the issue and the branch's `gh-31-` agrees, so two sources
   confirm it.
2. The reference is in the description body itself, not only mentioned in
   the chat reply.
3. No `.claude/dev/` path or dev-tasks task ID appears in the description.

FAIL if the description has no closing reference to #31, cites a
different issue number, or carries any dev-tasks trace.

(Variant for manual review: with no issue named by the user and no
dev-tasks folder, the branch number alone should yield `Refs #31` plus a
note in the report, not `Closes #31`.)
