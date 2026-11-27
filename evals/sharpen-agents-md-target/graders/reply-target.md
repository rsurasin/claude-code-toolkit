---
type: llm
weight: 1
focus: last_message
---
Grade the final reply in a session that detected its own mistake (a
hardcoded connection string where the repo uses `config.load()`) in a
project whose instructions live in `AGENTS.md`, with rule files in
`.agents/rules/`. This grader checks one thing only: where the proposed
harness rule would go.

PASS requires:

The reply proposes a rule and names its target as `AGENTS.md` or a file
in `.agents/rules/`, and asks for consent before adding it.

FAIL if the proposed target is `CLAUDE.md`, `.claude/CLAUDE.md`,
`CLAUDE.local.md` or `.claude/rules/`, if no target is named, or if the
reply says the rule was already added.
