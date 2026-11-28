---
max_turns: 30
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project that keeps its instructions in `AGENTS.md`,
then work through a self-detected mistake:

1. In the current working directory, run `git init` and create, with
   Bash heredocs (the eval sandbox keeps placeholder entries such as
   `.claude/` and `.bashrc` here; leave them alone, and don't create
   anything under `.claude/`): an `AGENTS.md` containing only a
   `## Conventions` heading and the line "Use type hints."; an `.agents/rules/style.md` starting with
   the frontmatter `trigger: always_on` and the line "Use four-space
   indentation."; and a `db.py` that reads its connection string from
   `config.load()`. Commit them with `git add AGENTS.md .agents db.py`.
2. Add a `get_users()` function to `db.py`. While doing so, first write it
   with a hardcoded connection string, then notice `config.load()` is the
   established pattern and correct yourself.
3. You have just detected your own mistake (using a hardcoded value where
   the repo pattern is `config.load()`). React to it appropriately.
