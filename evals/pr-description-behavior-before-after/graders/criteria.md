Grade the transcript against the pr-description skill's Behavior
Before / After rules.

PASS requires ALL of:

1. **Fix PR — Summary states the problem and current behavior.** It
   opens with what a caller sees today, in observable terms: a
   lowercase `save10` code is applied twice and a 100.00 cart is charged
   81.00 instead of 90.00 (exact numbers optional; the double discount
   on lowercase codes is required). It is readable by a PM (no code
   identifiers needed to understand it), then gives one sentence on the
   fix.
2. **Fix PR — Changes leads with the new behavior.** The Changes
   section opens with the new behavior in the same terms as the current
   behavior (e.g., `save10` and `SAVE10` both charge 90.00), so the two
   read as a before/after pair. Area-grouped implementation detail
   follows under it, not before it.
3. **Refactor PR — "No behavior change" in both places.** The Summary
   and the Changes section each say "no behavior change" (or a clear
   equivalent). No before/after behavior is invented for the refactor.
4. Both descriptions keep the rest of the template (Risk Assessment
   with an evidence table, Testing, Screenshots / Output, Notes for
   Reviewers, Checklist).

FAIL if the fix PR's Summary describes only the code change without
the current behavior, if its Changes section starts with code-level
items and never states the new behavior, or if the refactor PR claims
a behavior change or omits "no behavior change" from either section.
