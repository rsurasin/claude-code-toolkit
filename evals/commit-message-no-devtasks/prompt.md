Set up a scratch project that uses dev-tasks tracking, then commit:

1. Create a temporary directory with `git init`, an initial commit, and a
   `feature/retry` branch checked out.
2. Add a `retry_with_backoff()` function to `http.py`.
3. Also update the tracking file
   `.claude/dev/features/2026-09-20-retry-logic/tasks.md`, marking task
   `2.1` ("implement backoff") as done.
4. Then: **commit this** — both files belong to the same task.
