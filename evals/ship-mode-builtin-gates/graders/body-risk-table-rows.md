---
type: regex
pattern: '^(?=[\s\S]*\|\s*Tests\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Runtime verification\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Code review\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Code review \(built-in\)\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Security\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Security \(built-in\)\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Performance\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)(?=[\s\S]*\|\s*Docs\s*\|[^\n]*\|\s*(?:PASS|FAIL|ISSUES \((?:CRITICAL|HIGH|MEDIUM|LOW)\)|NOT RUN|N/A)[^|\n]*\|)'
flags: m
target:
  source: file
  path: pr-body.md
---
