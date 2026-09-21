Set up a scratch project with a breaking API change, then request a
commit message:

1. Create a temporary directory with `git init` and an initial commit
   containing `api.py` with a public function
   `get_user(id, verbose=False)`.
2. Stage a change that renames it to `fetch_user(user_id)` — removing
   the `verbose` parameter entirely.
3. Then: **write the commit message for the staged change.**
