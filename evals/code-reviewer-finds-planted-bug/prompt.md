Set up a scratch project with a planted bug, then request a review:

1. Create a temporary directory with `git init` and an initial commit
   containing `stats.py` with a working `mean(values)` function.
2. Commit a change adding `trimmed_mean(values, k)` that is intended to
   drop the k smallest and k largest values, but contains an off-by-one:
   it slices `sorted(values)[k:-k+1]` instead of `sorted(values)[k:-k]`
   (so for k=1 it keeps one of the largest values it should drop; note
   `[k:-k]` itself is also wrong when k=0 — the correct code must handle
   that too).
3. Then: **review my changes.**
