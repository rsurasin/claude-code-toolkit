---
type: regex
pattern: '\*\*Needs human review:?\*\*:?[^\n]*slug\.js'
match: not_contains
flags: i
target:
  source: file
  path: pr-body.md
---
