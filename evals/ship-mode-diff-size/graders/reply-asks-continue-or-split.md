---
type: regex
pattern: '(?:\bsplit[\s\S]{0,600}\b(?:continue|proceed|go ahead|carry on|keep going|as is|as one|single PR|one PR)\b|\b(?:continue|proceed|go ahead|carry on|keep going|as is|as one|single PR|one PR)\b[\s\S]{0,600}\bsplit)'
flags: i
target: last_message
---
