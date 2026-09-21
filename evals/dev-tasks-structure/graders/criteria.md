Grade the transcript against the dev-tasks skill's structure contract.

PASS requires ALL of:

1. A task folder was created at
   `.claude/dev/features/<YYYY-MM-DD>-<slug>/` — the `features` category
   (this is a new feature), a date prefix obtained via `date -u`
   (UTC, not a guessed or hardcoded date), and a kebab-case slug.
2. The folder contains all three files: `plan.md`, `context.md`, and
   `tasks.md`, each following the skill's documented format.
3. `.claude/dev/` is ensured present in the project's `.gitignore`, and
   the task files are NOT staged or committed.
4. No dev-tasks trace appears in any commit message made during the run.

FAIL if the category is wrong (e.g., dumped at `.claude/dev/<slug>`),
the date was not obtained via `date -u`, any of the three files is
missing, or the task files were committed.
