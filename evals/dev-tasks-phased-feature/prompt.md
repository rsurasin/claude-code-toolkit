---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project, then start planning a large feature:

1. In the current working directory (the eval sandbox keeps placeholder
   entries such as `.claude/` and `.bashrc` here; leave them alone), run
   `git init`, write a `.gitignore` containing `.claude/dev/`, and create
   an initial commit with a small Express app: `package.json`,
   `src/app.js` with one `GET /health` route, and `src/users.js` with an
   in-memory user store. Stage the files by name.
2. Then, as the user: **let's plan team accounts for the app and create
   the task documentation for it.** The design is settled:
   - a `teams` store and a `memberships` store (team, user, role);
   - inviting a user by email, with an invite token that expires;
   - roles `owner`, `admin`, `member`, with a permission check used by
     every team route;
   - REST routes to create a team, list members, invite, accept an
     invite, change a role, and remove a member;
   - tests for each of the above.
   I estimate about 1,300 changed lines across 24 files.
3. Write the `.claude/dev/` files with shell commands (e.g. heredocs);
   the sandbox may refuse the Write tool under `.claude/`. Don't start
   implementing.
4. End your reply with the `## Size` section of `plan.md` and the full
   `tasks.md`, each in its own fenced code block, exactly as you wrote
   them (or tried to, if the sandbox refused the write).
