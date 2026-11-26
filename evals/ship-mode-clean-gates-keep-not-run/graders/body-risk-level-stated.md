---
type: regex
pattern: '\*\*Risk Level:?\*{0,2}:?\s*(?:MEDIUM|HIGH|CRITICAL)\b'
flags: m
target:
  source: file
  path: pr-body.md
---
