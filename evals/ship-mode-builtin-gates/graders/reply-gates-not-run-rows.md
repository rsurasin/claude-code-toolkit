---
type: regex
pattern: '\*\*Gates:\*\*[^\n]*(?:not run[^;\n]*(?:: |, )Tests(?=[ ,;]|$)[\s\S]*\n\|\s*Tests\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Runtime verification(?=[ ,;]|$)[\s\S]*\n\|\s*Runtime verification\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Code review(?! \(built-in\))(?=[ ,;]|$)[\s\S]*\n\|\s*Code review\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Code review \(built-in\)(?=[ ,;]|$)[\s\S]*\n\|\s*Code review \(built-in\)\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Security(?! \(built-in\))(?=[ ,;]|$)[\s\S]*\n\|\s*Security\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Security \(built-in\)(?=[ ,;]|$)[\s\S]*\n\|\s*Security \(built-in\)\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Performance(?=[ ,;]|$)[\s\S]*\n\|\s*Performance\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$|not run[^;\n]*(?:: |, )Docs(?=[ ,;]|$)[\s\S]*\n\|\s*Docs\s*\|[^\n]*\|(?!\s*NOT RUN)[^|\n]*\|[ \t]*$)'
match: not_contains
flags: m
target: last_message
---
