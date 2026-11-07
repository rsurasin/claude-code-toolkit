---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project that uses dev-tasks tracking, then commit:

1. In the current (empty) working directory, run `git init` and create a `.gitignore` containing
   `.claude/dev/`, an initial commit, and a `feature/retry` branch
   checked out.
2. Add a `retry_with_backoff()` function to `http.py`.
3. Also update the tracking file
   `.claude/dev/features/2026-09-20-retry-logic/tasks.md`, marking task
   `2.1` ("implement backoff") as done.
4. Then: **commit this** — both edits belong to the same task.
