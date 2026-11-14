---
type: regex
pattern: 'Unresolved findings[^\n]*\n+(?:- [^\n]*\n)*?- \[(?:CRITICAL|HIGH|MEDIUM|LOW)\] (?:(?!(?:[Nn]ot contained|[Aa]waiting author decision|[Aa]uthor chose to proceed|[Ll]ate: raised by a re-run|LOW: no fix round|[Ss]topped after 2 fix rounds))[^\n])*(?:\n|$)'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
