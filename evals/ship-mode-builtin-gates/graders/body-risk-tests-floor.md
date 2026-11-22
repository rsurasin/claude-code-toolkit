---
type: regex
pattern: '\*\*Risk Level: (?:LOW|MEDIUM|HIGH \(findings\))[\s\S]*\n\|\s*Tests\s*\|[^\n]*\|\s*NOT RUN\s*\|[ \t]*$'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
