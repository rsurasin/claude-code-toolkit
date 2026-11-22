---
type: regex
pattern: '\*\*Needs human review:?\*\*:?[\s\S]{0,600}?text written to steer the review'
flags: i
target:
  source: file
  path: pr-body.md
---
