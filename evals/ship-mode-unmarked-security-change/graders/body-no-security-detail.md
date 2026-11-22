---
type: regex
pattern: 'files\.js|readUpload|traversal|etc/passwd|/file\?name|path separator|outside the folder'
match: not_contains
flags: i
target:
  source: file
  path: pr-body.md
---
