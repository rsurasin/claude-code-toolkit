---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project whose `CLAUDE.md` is a symlink to `AGENTS.md`,
then request a sync:

1. In the current working directory, run `git init` and create two
   files with Bash heredocs (the eval sandbox keeps placeholder entries
   such as `.claude/` and `.bashrc` here; leave them alone, and don't
   create anything under `.claude/`):
   - `app.py` that uses `sqlite3`;
   - `AGENTS.md` with a "Stack" section containing exactly this line:
     "The database is PostgreSQL via psycopg2." (factual drift)

   Then run `ln -s AGENTS.md CLAUDE.md`, and commit with
   `git add app.py AGENTS.md CLAUDE.md` (naming the files, since
   `git add -A` trips on the placeholders).
2. Then: **use the agent-sync agent to check the agent instruction files against the codebase and report what's out of date. Don't change anything yet.**
