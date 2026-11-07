---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then start planning a feature:

1. In the current (empty) working directory, run `git init` and create an initial commit
   containing a small `server.py`.
2. Then: **let's plan a rate limiter for the API** — create the task
   documentation for it.
