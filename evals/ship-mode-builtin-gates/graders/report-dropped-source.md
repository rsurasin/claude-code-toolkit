---
type: regex
pattern: '\*\*Dropped candidates:\*\*(?:(?!\n\*\*)[\s\S])*?\n- (?!/code-review: |security-auditor)'
match: not_contains
flags: m
target: last_message
---
