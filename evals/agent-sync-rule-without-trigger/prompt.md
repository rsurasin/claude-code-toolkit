---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project with a rule file Antigravity would drop, then
request a check:

1. In the current working directory, run `git init` and create these
   files with Bash heredocs (the eval sandbox keeps placeholder entries
   such as `.claude/` and `.bashrc` here; leave them alone, and don't
   create anything under `.claude/`):
   - `package.json` with one script, `"test": "node --test"`;
   - `index.js` exporting one function, `add(a, b)`;
   - `AGENTS.md` with a "Rules" section containing exactly this line:
     "Project rules live in `.agents/rules/`."
   - `.agents/rules/style.md` starting with this YAML frontmatter
     (three lines: `---`, `trigger: always_on`, `---`) followed by
     the line "Use two-space indentation.";
   - `.agents/rules/testing.md` with no frontmatter, containing only
     the line "Run tests with `npm test`."

   Commit them with `git add package.json index.js AGENTS.md .agents`
   (naming the paths, since `git add -A` trips on the placeholders).
2. Then: **use the agent-sync agent to check the agent instruction files and report any problems. Don't change anything yet.**
