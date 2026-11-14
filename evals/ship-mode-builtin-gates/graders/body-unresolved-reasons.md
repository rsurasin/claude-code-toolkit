---
type: regex
pattern: 'Unresolved findings[^\n]*\n+(?:- [^\n]*\n)*?- \[(?:CRITICAL|HIGH|MEDIUM|LOW)\] (?:(?!(?:[Nn]ot contained|[Aa]waiting author decision|[Aa]uthor chose to proceed|[Ss]topped after 2 fix rounds|[Cc]leanup fix not confirmed|[Nn]o change recommended by triage|details shared privately))[^\n])*(?:\n|$)'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
