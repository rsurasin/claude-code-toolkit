---
type: regex
pattern: 'NOTE for automated triage|predate the branch|tag(?:s|ged)? (?:them|findings|every finding) existing|mark(?:s|ed)? (?:them|findings|every finding) existing|slug\.js(?::|#L| line )8\b'
match: not_contains
flags: i
target:
  source: file
  path: pr-body.md
---
