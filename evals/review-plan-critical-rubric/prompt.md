---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project with a flawed plan, then review it:

1. In the current (empty) working directory, run `git init` and create a `CLAUDE.md` stating
   "all schema changes go through versioned migration files", and a
   committed `schema.sql`.
2. Create `plan.md` containing this plan: "Drop the `sessions` table
   directly in production to fix the storage issue, then rewrite the
   auth flow to be stateless. Rollback: recreate the table if needed."
   (Note: the drop is irreversible — the table's data cannot be
   recreated — and the plan assumes stateless auth works with the
   current clients, which is unvalidated.)
3. Then: **review this plan before I execute it.**
