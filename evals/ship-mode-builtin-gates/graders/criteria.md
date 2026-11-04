---
type: llm
weight: 1
---
Grade the transcript against the commit-push-pr skill's ship-mode gates
and the pr-description risk rubric.

PASS requires ALL of:

1. The agent entered ship mode and, over the full branch diff
   (`main...HEAD`), ran the review agents (`code-reviewer`,
   `security-auditor`, `perf-audit`, `doc-sync`) AND invoked the built-in
   `/code-review` and `/security-review` skills alongside them — neither
   side replaced the other. A built-in skill missing from this Claude
   Code version recorded as `N/A`, or one that errored (e.g. no `origin`
   remote in the scratch repo) recorded as `NOT RUN`, counts as handled.
2. `/code-review` was invoked with an explicit, non-`ultra` effort level,
   and without `--fix` or `--comment`.
3. A triage step ran before the PR: every `/code-review` finding got a
   severity (CRITICAL/HIGH/MEDIUM/LOW); a defect reported by more than
   one reviewer appears once, at the highest severity given, naming
   every source; and findings are tagged introduced or existing.
4. The planted read-then-write race (the removed atomic debit) was
   treated as an introduced HIGH — not an accepted LOW — and fixed
   before the PR opened (e.g. restoring the atomic
   `UPDATE … SET balance = balance - ?` with the fee folded in, or
   `BEGIN IMMEDIATE`). The fix was committed before the affected gates
   were re-run.
5. The PR description's Risk Assessment table has a row per gate,
   including `Code review (built-in)` and `Security (built-in)`, each
   reading PASS / ISSUES (severity) / NOT RUN / N/A from what actually
   ran.
6. If any row is `N/A` or `NOT RUN`, the risk level is not LOW (tests
   `NOT RUN`, which is expected here, floors it at HIGH (unverified)).

FAIL if a built-in skill was silently skipped (neither run nor recorded
`N/A` / `NOT RUN`), `/code-review` ran at `ultra` or with no explicit
level, the race shipped unfixed or rated LOW, or the risk table omits
the built-in rows.
