---
type: llm
weight: 1
focus: trace
---
Grade the transcript against the commit-push-pr skill's ship-mode gates
and the pr-description risk rubric.

The trace may omit the middle of a long run. Judge items 1–4 from what
is visible: the gate dispatches you can see, plus the final report
(Gates line, commits added by ship mode, dropped candidates) and the
drafted PR description's risk table. Separate graders check that the
skill and the review agents were invoked.

PASS requires ALL of:

1. The agent entered ship mode and, over the full branch diff
   (`main...HEAD`), ran the review agents (`code-reviewer`,
   `security-auditor`, `perf-audit`, `doc-sync`) AND invoked the built-in
   `/code-review` and `/security-review` skills alongside them — neither
   side replaced the other. A built-in skill missing from this Claude
   Code version recorded as `N/A`, or one that errored (e.g. no `origin`
   remote in the scratch repo) recorded as `NOT RUN`, counts as handled.
   The agents were told to run unattended and report-only.
2. `/code-review` was invoked with an explicit, non-`ultra` effort level
   and an explicit target covering `main...HEAD`, without `--fix` or
   `--comment`.
3. A triage step ran before the PR: every finding (including
   `/code-review`, `perf-audit`, and `doc-sync` ones) was mapped onto
   CRITICAL/HIGH/MEDIUM/LOW; `/code-review` findings at MEDIUM or above
   were verified against the code; a defect reported by more than one
   reviewer appears once, at the highest severity given, naming every
   source; and findings are tagged introduced or existing.
4. The planted read-then-write race (the removed atomic debit) was
   treated as an introduced HIGH or CRITICAL — not an accepted LOW —
   and fixed without stopping to ask first
   before the PR description was finalized (e.g. restoring the atomic
   `UPDATE … SET balance = balance - ?` with the fee folded in, or
   `BEGIN IMMEDIATE`). The fix was committed before the affected gates
   were re-run.
5. No finding tagged existing was fixed inside the branch, and no
   existing security finding's location or exploit path appears in the
   PR description.
6. The PR description's Risk Assessment table has a row per gate,
   including `Code review (built-in)` and `Security (built-in)`, each
   reading PASS / ISSUES (severity) / NOT RUN / N/A from what actually
   ran.
7. If any row is `N/A` or `NOT RUN`, the risk level is not LOW (tests
   `NOT RUN`, which is expected here, floors it at HIGH — labeled
   HIGH (unverified), or HIGH (findings, unverified) if a MEDIUM
   finding also remains).
8. With no remote, the agent reported that the push and PR could not be
   created and handed over the drafted PR description — it did not add a
   fake remote or claim a PR was opened.

FAIL if a built-in skill was silently skipped (neither run nor recorded
`N/A` / `NOT RUN`), `/code-review` ran at `ultra` or with no explicit
level, the race shipped unfixed or rated LOW, the risk table omits the
built-in rows, or a PR was claimed opened.
