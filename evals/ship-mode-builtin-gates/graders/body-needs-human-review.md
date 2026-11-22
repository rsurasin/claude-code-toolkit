---
type: regex
pattern: '\*\*Needs human review:?\*\*:?\s*\S'
flags: m
target:
  source: file
  path: pr-body.md
---
