---
type: regex
pattern: '^\*\*Needs human review:?\*\*:?[^\n]*AGENTS\.md'
flags: m
target:
  source: file
  path: pr-body.md
---
