---
type: regex
pattern: 'Needs human review[\s\S]{0,800}?\b(base|default branch)\b'
flags: i
target:
  source: file
  path: pr-body.md
---
