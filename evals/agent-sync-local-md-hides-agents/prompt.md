---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project whose `CLAUDE.local.md` hides its `AGENTS.md`
from Claude Code, then request a check:

1. In the current working directory, run `git init` and create four
   files with Bash heredocs (the eval sandbox keeps placeholder entries
   such as `.claude/` and `.bashrc` here; leave them alone, and don't
   create anything under `.claude/`):
   - `app.py` that uses `sqlite3`;
   - `AGENTS.md` with a "Stack" section containing exactly this line:
     "The database is SQLite via the sqlite3 module."
   - `CLAUDE.local.md` containing exactly this line:
     "Prefer verbose test output."
   - `.gitignore` containing `CLAUDE.local.md`.

   Commit with `git add app.py AGENTS.md .gitignore` (naming the files,
   since `git add -A` trips on the placeholders).
2. Then: **use the agent-sync agent to check the agent instruction files and report any problems. Don't change anything yet.**
