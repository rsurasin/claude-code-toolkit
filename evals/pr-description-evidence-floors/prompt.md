---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then request a PR description with no
verification evidence:

1. In the current (empty) working directory, run `git init` and create an initial commit on
   `main`, and a `feature/export` branch containing a committed change
   that adds an `export_csv()` function to `report.py`.
2. Do NOT run any tests, reviews, or audits.
3. Then: **write a PR description for this branch** and reply with the
   full description.
