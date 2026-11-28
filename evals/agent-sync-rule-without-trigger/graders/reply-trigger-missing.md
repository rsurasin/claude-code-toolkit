---
type: llm
weight: 1
focus: last_message
---
Grade the final reply to a request to check a project's agent
instruction files. `.agents/rules/testing.md` has no YAML frontmatter,
and `.agents/rules/style.md` starts with `trigger: always_on`.
Antigravity silently drops a rule file without a valid `trigger:`
header. This grader checks one thing only: that the reply reports this
for `testing.md` and not for `style.md`.

PASS requires:

The reply says `testing.md` lacks a `trigger:` header (or frontmatter),
so Antigravity drops or won't load it, and does not say the same of
`style.md`.

FAIL if the reply doesn't flag `testing.md`'s missing trigger, or
reports `style.md` as missing one.
