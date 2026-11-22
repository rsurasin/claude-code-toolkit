---
type: llm
weight: 1
focus: last_message
---
Grade the final reply to a request to update a project's agent
instruction files. The project has a separate `CLAUDE.md` (not a
symlink, and not an `@AGENTS.md` import) beside an `AGENTS.md` with
different content. This grader checks one thing only: that the reply
flags the two separate files as a setup problem.

PASS requires:

The reply says that, with a separate `CLAUDE.md` present, Claude Code
reads `CLAUDE.md` and does not read `AGENTS.md` by default (in any
wording, e.g. "Claude Code ignores AGENTS.md here").

FAIL if the reply does not say this.
