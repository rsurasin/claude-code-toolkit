---
type: llm
weight: 1
focus: last_message
---
Grade the final reply to a request to check a project's agent
instruction files. The project's `AGENTS.md` is about 4k tokens, over
the ~3k budget, and other harnesses read it, so content split out of it
belongs in `.agents/rules/` (or a nested `AGENTS.md`), not
`.claude/rules/`, which only Claude Code reads. This grader checks one
thing only: where the reply suggests moving content.

PASS requires:

The reply flags `AGENTS.md` as over budget and suggests moving sections
into `.agents/rules/` files (a nested `AGENTS.md` is also fine), and
does not suggest moving them into `.claude/rules/`.

FAIL if it suggests `.claude/rules/` as the destination, doesn't flag
the size, or suggests no destination.
