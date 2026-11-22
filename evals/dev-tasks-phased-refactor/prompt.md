---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then start planning a large refactor:

1. In the current working directory (the eval sandbox keeps placeholder
   entries such as `.claude/` and `.bashrc` here; leave them alone), run
   `git init`, write a `.gitignore` containing `.claude/dev/`, and create
   an initial commit with a small Node app whose modules use callbacks:
   `package.json`, `src/db.js` with a `query(sql, params, callback)`
   helper, and `src/orders.js` and `src/invoices.js`, each calling
   `db.query` with a callback. Stage the files by name.
2. Then, as the user: **let's plan a refactor and create the task
   documentation for it: move every module from callbacks to
   async/await, and replace the `db.query` calls with one repository
   module per table.** No behavior changes. The real app has many more
   modules than this sample, so I estimate about 900 changed lines
   across 30 files.
3. Write the `.claude/dev/` files with shell commands (e.g. heredocs);
   the sandbox may refuse the Write tool under `.claude/`. Don't start
   implementing.
4. End your reply with the `## Size` section of `plan.md` and the full
   `tasks.md`, each in its own fenced code block, exactly as you wrote
   them (or tried to, if the sandbox refused the write).
