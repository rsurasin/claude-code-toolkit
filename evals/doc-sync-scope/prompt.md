Set up a scratch project with drifted docs inside and outside doc-sync's
scope, then request a sync:

1. Create a temporary directory with `git init` and an initial commit
   containing: `lib.py` with a function `fetch_records()`; `README.md`
   documenting it under its old name `get_records()` (drift); and
   `.claude/rules/data.md` also referencing `get_records()` (drift, but
   out of doc-sync's scope).
2. Then: **sync the documentation with the current code.**
