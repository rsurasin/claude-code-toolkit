---
type: regex
pattern: '^\|\s*Security \(built-in\)\s*\|[^\n]*\|\s*NOT RUN\b'
flags: m
target:
  source: file
  path: pr-body.md
---
