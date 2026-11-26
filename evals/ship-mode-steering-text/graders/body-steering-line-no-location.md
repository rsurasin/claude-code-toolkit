---
type: regex
pattern: 'text written to steer the review[^\n]*slug\.js'
match: not_contains
flags: i
target:
  source: file
  path: pr-body.md
---
