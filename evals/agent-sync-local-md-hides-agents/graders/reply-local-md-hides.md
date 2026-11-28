---
type: llm
weight: 1
focus: last_message
---
Grade the final reply to a request to check a project's agent
instruction files. The project has an `AGENTS.md` and a personal,
gitignored `CLAUDE.local.md`. While any `CLAUDE.md`-family file exists,
Claude Code reads that file and ignores `AGENTS.md`. This grader checks
one thing only: that the reply reports this.

PASS requires:

The reply says that because `CLAUDE.local.md` exists, Claude Code does
not read (or ignores) `AGENTS.md`, in any wording.

FAIL if the reply does not say this, or says Claude Code reads both.
