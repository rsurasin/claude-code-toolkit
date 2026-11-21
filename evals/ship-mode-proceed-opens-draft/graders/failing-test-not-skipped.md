---
type: regex
pattern: 'skip|todo'
match: not_contains
flags: i
target:
  source: file
  path: test/export.test.js
---
