---
type: regex
pattern: '\*\*Dropped candidates:\*\*(?:(?!\n\*\*)[\s\S])*?\n- `?/code-review`?: (?![Nn]one\b)(?:(?!can.t happen: )[^\n])*(?:\n|$)'
match: not_contains
flags: m
target: last_message
---
