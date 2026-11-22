---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project whose agent instruction files disagree, then
request a sync:

1. In the current working directory, run `git init` and create four
   files with Bash heredocs (the eval sandbox keeps placeholder entries
   such as `.claude/` and `.bashrc` here; leave them alone, and don't
   create anything under `.claude/`):
   - `package.json` with two scripts, `"test": "node --test"` and
     `"check": "node --check index.js && node --test"`;
   - `index.js` exporting one function, `add(a, b)`;
   - `AGENTS.md` with a "Commands" section containing exactly this line:
     "Before committing, run `npm run check`."
   - `CLAUDE.md`, a separate file (not a symlink), with a "Commands"
     section containing exactly this line:
     "Before committing, run `npm test`."

   Commit them with `git add package.json index.js AGENTS.md CLAUDE.md`
   (naming the files, since `git add -A` trips on the placeholders).
2. Then: **update the agent instruction files to match the codebase.**
