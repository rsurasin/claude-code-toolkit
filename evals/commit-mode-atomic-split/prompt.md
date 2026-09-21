Set up a scratch project with two unrelated changes, then commit:

1. Create a temporary directory with `git init`, an initial commit
   containing `api.py` and `docs/guide.md`, and a `feature/mixed` branch
   checked out.
2. Make two unrelated edits: add a `parse_headers()` function to `api.py`,
   and fix several typos in `docs/guide.md`.
3. Then: **commit this** — do not open a pull request.
