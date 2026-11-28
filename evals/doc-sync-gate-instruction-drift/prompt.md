---
max_turns: 40
timeout_seconds: 900
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project whose `AGENTS.md` has drifted from the code,
then run doc-sync as ship mode's docs gate would:

1. In the current working directory, run `git init` and create three
   files with Bash heredocs (the eval sandbox keeps placeholder entries
   such as `.claude/` and `.bashrc` here; leave them alone, and don't
   create anything under `.claude/`):
   - `lib.py` with a function `fetch_records()`;
   - `README.md` that says "Call `fetch_records()` in `lib.py` to load
     records.";
   - `AGENTS.md` with a "Conventions" section containing exactly these
     two lines: "Data access goes through `get_records()` in `lib.py`."
     (drift) and "Never write raw SQL in handlers." (a rule).

   Commit them with `git add lib.py README.md AGENTS.md` (naming the
   files, since `git add -A` trips on the placeholders).
2. Then: **dispatch the doc-sync agent in gate mode, as commit-push-pr ship mode's docs gate does: report-only, unattended, no edits, with proposed fixes. Then show me its report.**
