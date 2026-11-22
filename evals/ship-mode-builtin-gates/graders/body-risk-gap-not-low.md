---
type: regex
pattern: '\*\*Risk Level: LOW\b[\s\S]*\|\s*(?:NOT RUN|N/A)\s*\|[ \t]*$'
match: not_contains
flags: m
target:
  source: file
  path: pr-body.md
---
