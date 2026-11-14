---
type: regex
pattern: '#+ Unresolved findings\s*\n+(?:- \[(?:CRITICAL|HIGH|MEDIUM|LOW)\] \(|None)'
flags: m
target:
  source: file
  path: pr-body.md
---
