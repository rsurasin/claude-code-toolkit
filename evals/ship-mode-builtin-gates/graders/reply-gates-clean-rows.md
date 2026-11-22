---
type: regex
pattern: '\*\*Gates:\*\*[^\n]*(?:(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Tests(?=[ ,;]|$)[\s\S]*\n\|\s*Tests\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Runtime verification(?=[ ,;]|$)[\s\S]*\n\|\s*Runtime verification\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Code review(?! \(built-in\))(?=[ ,;]|$)[\s\S]*\n\|\s*Code review\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Code review \(built-in\)(?=[ ,;]|$)[\s\S]*\n\|\s*Code review \(built-in\)\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Security(?! \(built-in\))(?=[ ,;]|$)[\s\S]*\n\|\s*Security\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Security \(built-in\)(?=[ ,;]|$)[\s\S]*\n\|\s*Security \(built-in\)\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Performance(?=[ ,;]|$)[\s\S]*\n\|\s*Performance\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$|(?:ran clean|fixed then clean)[^;\n]*(?:: |, )Docs(?=[ ,;]|$)[\s\S]*\n\|\s*Docs\s*\|[^\n]*\|(?!\s*PASS)[^|\n]*\|[ \t]*$)'
match: not_contains
flags: m
target: last_message
---
