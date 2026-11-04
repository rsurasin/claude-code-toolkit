---
type: llm
weight: 1
focus: last_message
---

Grade the agent's final reply against the dev-tasks skill's structure
contract. Separate graders check that `date -u` ran, which files were
written, and that no task file was staged.

Note: the eval sandbox refuses writes under `.claude/`. If the reply says
the task files were refused for that reason, that is the environment,
not a failure — grade the folder and files the agent attempted.

PASS requires ALL of:

1. The reply names a task folder at
   `.claude/dev/features/<YYYY-MM-DD>-<slug>/` — the `features` category
   (this is a new feature) and a kebab-case slug — not
   `.claude/dev/<slug>/` or another category.
2. The folder holds (or the agent attempted) all three files: `plan.md`,
   `context.md`, and `tasks.md`.
3. `.claude/dev/` is ensured present in the project's `.gitignore`, and
   the task files are not staged or committed.
4. No dev-tasks trace appears in any commit message the reply reports.

FAIL if the category is wrong, any of the three files is missing from
the plan, or the task files were committed.
