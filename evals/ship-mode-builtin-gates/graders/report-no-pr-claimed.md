---
type: regex
pattern: 'github\.com/[\w.-]+/[\w.-]+/pull/\d+'
match: not_contains
flags: m
target: last_message
---
