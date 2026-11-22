---
type: regex
pattern: '^\|\s*Performance\s*\|[^\n]*\|\s*NOT RUN\b'
flags: m
target:
  source: file
  path: pr-body.md
---
