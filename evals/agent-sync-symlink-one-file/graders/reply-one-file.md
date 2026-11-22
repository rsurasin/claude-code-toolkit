---
type: llm
weight: 1
focus: last_message
---
Grade the final reply to a request to update a project's agent
instruction files. The project's `CLAUDE.md` is a symlink to
`AGENTS.md`, so the two names are one file. This grader checks one
thing only: that the reply treats them as one file.

PASS requires:

The reply does not report `CLAUDE.md` and `AGENTS.md` as contradicting,
duplicating, or drifting from each other, and does not warn that Claude
Code ignores `AGENTS.md`. Saying that `CLAUDE.md` is a symlink to
`AGENTS.md`, or listing the database drift once, is fine.

FAIL if the reply reports a conflict or difference between the two
names, lists the same drift twice as if in two files, or warns that
`AGENTS.md` is ignored.
