---
type: regex
pattern: '\*\*Risk Level: (?:LOW|MEDIUM|HIGH \((?:unverified|findings|findings, unverified)\)|CRITICAL)\*\*'
flags: m
target:
  source: file
  path: pr-body.md
---
