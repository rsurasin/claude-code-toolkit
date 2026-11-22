---
type: regex
pattern: '^\|\s*Code review\s*\|[^\n]*\|\s*PASS\b'
flags: m
target:
  source: file
  path: pr-body.md
---
