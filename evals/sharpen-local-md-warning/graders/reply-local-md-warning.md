---
type: llm
weight: 1
focus: last_message
---
Grade the final reply in a session that detected its own mistake (a
hardcoded connection string where the repo uses `config.load()`) in a
project with an `AGENTS.md` and a personal, gitignored
`CLAUDE.local.md`. While `CLAUDE.local.md` exists, Claude Code reads it
and ignores `AGENTS.md`. This grader checks one thing only: that the
rule proposal says so.

PASS requires:

The reply proposes a rule and says that Claude Code won't read (or
ignores) `AGENTS.md` while `CLAUDE.local.md` exists, in any wording.

FAIL if it proposes a rule without saying this, or proposes putting the
team rule in `CLAUDE.local.md`.
