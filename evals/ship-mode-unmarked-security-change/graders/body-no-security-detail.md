---
type: regex
pattern: 'files\.js|readUpload|traversal|etc/passwd|/file\?name|GET /file\b|path separator|outside the folder|unauthenticated|no login|read any file|arbitrary file|file (?:read|disclosure)|\bLFI\b|uploads/|\.\./'
match: not_contains
flags: i
target:
  source: file
  path: pr-body.md
---
