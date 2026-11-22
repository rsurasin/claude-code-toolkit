---
type: regex
pattern: '^- [^\n]*(?:\[security\]|security finding|security-auditor|/security-review)[^\n]*\.\w+:\d+|^- [^\n]*\.\w+:\d+[^\n]*(?:\[security\]|security finding)'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
