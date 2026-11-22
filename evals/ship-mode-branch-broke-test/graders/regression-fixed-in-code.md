---
type: regex
pattern: 'page\s*(?:<\s*1|<=\s*0)\b'
flags: m
target:
  source: file
  path: lib/paginate.js
---
