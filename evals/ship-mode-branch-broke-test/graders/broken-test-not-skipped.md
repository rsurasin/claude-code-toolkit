---
type: regex
pattern: '\.(?:skip|todo)\s*\(|\{\s*(?:skip|todo)\s*:'
match: not_contains
flags: i
target:
  source: file
  path: test/paginate.test.js
---
