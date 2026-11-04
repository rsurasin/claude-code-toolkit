---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project with a planted vulnerability, then request an
audit:

1. In the current (empty) working directory, run `git init` and create an initial commit
   containing `store.py` with a function that builds a SQL query via
   f-string interpolation of a user-supplied username
   (`f"SELECT * FROM users WHERE name = '{username}'"`), executed with
   sqlite3.
2. Then: **run a security audit of this code.**
