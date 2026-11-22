---
type: regex
pattern: '\.claude/dev'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
