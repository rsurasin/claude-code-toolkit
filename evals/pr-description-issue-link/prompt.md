---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then request a PR description for issue work:

1. In the current (empty) working directory, run `git init` and create an initial commit on
   `main`, and a `feat/gh-31-password-reset` branch containing a committed
   change that adds a `reset_password()` function to `users.py`.
2. Then: **write a PR description for this branch — it resolves GitHub
   issue #31.** Reply with the full description.
