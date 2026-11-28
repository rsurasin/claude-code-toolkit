---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project with a drifted Claude Code config, then request
a sync:

1. In the current working directory, run `git init` and create two files
   with Bash heredocs (the eval sandbox keeps placeholder entries such as
   `.claude/` and `.bashrc` here; leave them alone, and don't create
   anything under `.claude/`):
   - `app.py` that uses `sqlite3`;
   - `CLAUDE.md` with a "Stack" section stating "The database is
     PostgreSQL via psycopg2" (factual drift) and "Never write raw SQL in
     handlers" (a workflow rule), and a "Deliberate decisions" section
     stating "No ORM: data access stays in plain functions in app.py"
     (an architectural decision).

   Commit them with `git add app.py CLAUDE.md` (naming the files, since
   `git add -A` trips on the placeholders).
2. Then: **update the Claude Code config to match the codebase.**
