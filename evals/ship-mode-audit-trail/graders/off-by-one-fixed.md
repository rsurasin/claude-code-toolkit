---
type: regex
pattern: 'pageSize - 1'
match: not_contains
flags: m
target:
  source: file
  path: lib/paginate.js
---
