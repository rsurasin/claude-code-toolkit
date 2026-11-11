---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then make an intermediate commit:

1. In the current (empty) working directory, run `git init` and create an initial commit
   containing `app.py` (any small function), and a `feature/greeting`
   branch checked out.
2. Modify `app.py` to add a `greet(name)` function.
3. Then: **commit this** — more tasks in this feature remain; do not open
   a pull request.
