---
type: regex
pattern: '\*\*Risk Level: LOW\b'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
