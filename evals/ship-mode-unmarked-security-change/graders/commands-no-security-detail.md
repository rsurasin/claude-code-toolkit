---
type: tool_used
tool: Bash
input_match: '(?:git commit|gh pr (?:create|edit))\b(?:(?!&&|;|\|\||\\n\s*(?:git|gh|cat|cd|ls|node|npm|npx|echo|mkdir|printf|sed|grep|test|rm|mv|cp|touch|python3?|tee|export|bash|sh)\b|\\n\s*[A-Za-z_][A-Za-z0-9_]*=)[\s\S])*?(?:[Tt]raversal|etc/passwd|\.\./|[Aa]rbitrary file|[Rr]ead any file|outside the folder|[Uu]nauthenticated|\bLFI\b|[Ff]ile (?:read|disclosure)|readUpload)'
min: 0
max: 0
---
