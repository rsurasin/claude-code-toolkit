Set up a scratch project with a planted vulnerability, then request an
audit:

1. Create a temporary directory with `git init` and an initial commit
   containing `store.py` with a function that builds a SQL query via
   f-string interpolation of a user-supplied username
   (`f"SELECT * FROM users WHERE name = '{username}'"`), executed with
   sqlite3.
2. Then: **run a security audit of this code.**
