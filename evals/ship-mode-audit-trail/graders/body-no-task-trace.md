---
type: regex
pattern: '\.claude/dev|Review Fixes|AUDIT \(|gh-7-page-size/'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
