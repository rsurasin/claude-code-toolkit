Set up a scratch project, then work through a self-detected mistake:

1. Create a temporary directory with `git init`, a `CLAUDE.md` containing
   a "Conventions" section, and a committed `db.py` that reads its
   connection string from `config.load()`.
2. Add a `get_users()` function to `db.py`. While doing so, first write it
   with a hardcoded connection string, then notice `config.load()` is the
   established pattern and correct yourself.
3. You have just detected your own mistake (using a hardcoded value where
   the repo pattern is `config.load()`). React to it appropriately.
