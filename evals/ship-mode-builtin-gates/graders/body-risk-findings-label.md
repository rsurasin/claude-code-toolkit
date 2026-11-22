---
type: regex
pattern: '\*\*Risk Level: (?:LOW|MEDIUM|HIGH \(unverified\))[\s\S]*Unresolved findings[^\n]*\n+(?:- [^\n]*\n)*?- \[(?:MEDIUM|HIGH|CRITICAL)\]'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
