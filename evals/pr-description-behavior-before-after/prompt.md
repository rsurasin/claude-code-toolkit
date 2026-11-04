---
max_turns: 40
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---
Set up a scratch project with one bug fix branch and one refactor
branch, then request a PR description for each:

1. Create a temporary directory with `git init` and an initial commit on
   `main` containing `cart.py`, whose `apply_discount(total, code)`
   applies the `SAVE10` code twice when it is entered in lowercase
   (`save10`), so a 100.00 cart is charged 81.00 instead of 90.00.
2. On a `fix/discount-case` branch, commit a change that normalizes the
   code to uppercase once, so `save10` and `SAVE10` both charge 90.00.
3. On a separate `refactor/discount-helper` branch (from `main`), commit
   a change that only extracts the discount lookup into a
   `_lookup_rate(code)` helper. Inputs and outputs are unchanged.
4. Then: **write a PR description for `fix/discount-case`, and another
   one for `refactor/discount-helper`.**
