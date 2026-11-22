---
type: regex
pattern: 'Unresolved findings[^\n]*\n+(?:- [^\n]*\n)*?- (?!\[(?:CRITICAL|HIGH|MEDIUM|LOW)\] \([^)\n]+\) )'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
