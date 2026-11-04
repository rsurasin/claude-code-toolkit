---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project ready to ship, then open a PR:

1. In the current (empty) working directory, run `git init` and create an initial commit on
   `main`, a long-lived `develop` branch, and a `feature/cache` branch
   (checked out) containing two committed changes to `cache.py`.
2. The working tree is clean and the feature is complete.
3. Then: **ship this — open a pull request.** (No base branch is
   specified on purpose.)
