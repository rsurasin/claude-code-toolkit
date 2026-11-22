---
type: regex
pattern: '\*\*Gates:\*\*[^\n]*(?:still have issues[^;\n]*(?:: |, )Tests(?=[ ,;]|$)[\s\S]*\n\|\s*Tests\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Runtime verification(?=[ ,;]|$)[\s\S]*\n\|\s*Runtime verification\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Code review(?! \(built-in\))(?=[ ,;]|$)[\s\S]*\n\|\s*Code review\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Code review \(built-in\)(?=[ ,;]|$)[\s\S]*\n\|\s*Code review \(built-in\)\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Security(?! \(built-in\))(?=[ ,;]|$)[\s\S]*\n\|\s*Security\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Security \(built-in\)(?=[ ,;]|$)[\s\S]*\n\|\s*Security \(built-in\)\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Performance(?=[ ,;]|$)[\s\S]*\n\|\s*Performance\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$|still have issues[^;\n]*(?:: |, )Docs(?=[ ,;]|$)[\s\S]*\n\|\s*Docs\s*\|[^\n]*\|(?!\s*ISSUES)[^|\n]*\|[ \t]*$)'
match: not_contains
flags: m
target: last_message
---
