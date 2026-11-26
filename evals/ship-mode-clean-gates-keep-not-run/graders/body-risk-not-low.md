---
type: regex
pattern: '\*\*Risk Level:?\*{0,2}:?\s*LOW\b'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
