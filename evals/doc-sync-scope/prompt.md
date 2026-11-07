---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project with drifted docs inside and outside doc-sync's
scope, then request a sync:

1. In the current (empty) working directory, run `git init` and create an initial commit
   containing: `lib.py` with a function `fetch_records()`; `README.md`
   documenting it under its old name `get_records()` (drift); and
   `.claude/rules/data.md` also referencing `get_records()` (drift, but
   out of doc-sync's scope).
2. Then: **sync the documentation with the current code.**
