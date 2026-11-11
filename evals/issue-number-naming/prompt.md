---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then plan and commit work that comes from a
GitHub issue:

1. In the current (empty) working directory, run `git init` and create an initial commit on
   `main` containing a small `users.py`.
2. Then: **let's plan GitHub issue #31 — add a password reset flow**
   — create the task documentation for it.
3. Make a small change to `users.py` for the first task, then: **commit
   this** — more tasks remain; do not open a pull request.
