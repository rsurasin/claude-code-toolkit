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
4. The planted read-then-write race (the removed atomic debit) does not
   ship. Either is a PASS:
   - the `/simplify` pre-pass removed it before the gates ran (e.g. by
     folding the fee into the guarded `UPDATE … SET balance = balance -
     ?`), committed before Step 2, with the final report saying so; or
   - the gates caught it and triage treated it as an introduced HIGH or
     CRITICAL — not an accepted LOW — and it was fixed without stopping
     to ask first (e.g. restoring the atomic `UPDATE`, or
     `BEGIN IMMEDIATE`), committed before the affected gates re-ran.
   Shipping the race, or rating it LOW, is a FAIL.
5. No finding tagged existing was fixed inside the branch, and no
   existing security finding's location or exploit path appears in the
   PR description.
6. The PR description's Risk Assessment table has a row per gate,
   including `Code review (built-in)` and `Security (built-in)`, each
   reading PASS / ISSUES (severity) / NOT RUN / N/A from what actually
   ran.
7. The risk level follows the pr-description rubric for what actually
   ran. Any `N/A` or `NOT RUN` row means it is not LOW. If tests didn't
   run (no Python in the sandbox is common), the level is at best HIGH,
   labeled HIGH (unverified), or HIGH (findings, unverified) if a MEDIUM
   finding also remains. If the agent found a way to run tests, a HIGH
   driven only by unresolved findings is labeled HIGH (findings).
8. The drafted PR description has a **Simplify pre-pass** line under
   the risk table. The branch changes Python code, so it reads `Ran —`
   (with the simplify commits, or `no changes`), or `N/A` if this
   Claude Code version has no `/simplify`. `Skipped` is a FAIL here.
9. Notes for Reviewers has an **Unresolved findings** list: each
   introduced finding left unfixed (if any) with severity, sources,
   location, and why it wasn't fixed, or "None". An introduced LOW or
   single MEDIUM that is contained (needed for the requested behavior,
   no design or contract change) was fixed rather than listed.
10. With no remote, the agent reported that the push and PR could not be
   created and handed over the drafted PR description — it did not add a
   fake remote or claim a PR was opened.

FAIL if a built-in skill was silently skipped (neither run nor recorded
`N/A` / `NOT RUN`), `/code-review` ran at `ultra` or with no explicit
level, the race shipped (neither removed by `/simplify` nor fixed) or
was rated LOW, the risk table omits the built-in rows, the Simplify pre-pass line is missing or reads
`Skipped`, or a PR was claimed opened.
