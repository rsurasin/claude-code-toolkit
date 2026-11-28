---
type: regex
pattern: '^\*\*Needs human review:?\*\*:?(?:(?!\n#)[\s\S]){0,800}?AGENTS\.md'
flags: m
target:
  source: file
  path: pr-body.md
---
