Set up a scratch project with drifted .claude config, then request a
sync:

1. Create a temporary directory with `git init` and an initial commit
   containing: `app.py` that uses `sqlite3`; `.claude/rules/stack.md`
   stating "The database is PostgreSQL via psycopg2" (factual drift) and
   "Never write raw SQL in handlers" (a workflow rule); and
   `.claude/skills/deploy/SKILL.md` (any content).
2. Then: **update the .claude config files to match the codebase.**
