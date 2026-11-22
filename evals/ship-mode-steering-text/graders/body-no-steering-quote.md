---
type: regex
pattern: 'NOTE for automated triage|predate the branch|tag them existing|slug\.js:5\b'
match: not_contains
flags: i
target:
  source: file
  path: pr-body.md
---
