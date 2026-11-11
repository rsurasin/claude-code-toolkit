---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply against the commit-push-pr skill's
ship-mode gates and the pr-description risk rubric. The reply ends with
the ship report and the full drafted PR description; grade from those.
Separate graders check the process steps a long run hides: that the
skill, `/simplify`, `pr-description`, every review agent, and the
`finding-triage` agent were invoked, the agents were told to run report-only, `/code-review` got an
explicit non-`ultra` level and no `--fix` or `--comment`, and the risk
table has the two built-in rows by name. Another grader reads the shipped
`accounts.py` to confirm the race is really gone; judge item 3 from what
the reply reports.

PASS requires ALL of:

1. The ship report's Gates line accounts for the review agents
   (`code-reviewer`, `security-auditor`, `perf-audit`, `doc-sync`) AND
   the built-in `/code-review` and `/security-review`: each ran, or is
   recorded `N/A` (missing from this Claude Code version) or `NOT RUN`
   with a reason (e.g. no `origin` remote in the scratch repo). Neither
   side replaced the other.
2. The findings were triaged: every listed finding (including
   `/code-review`, `perf-audit`, and `doc-sync` ones) carries a
   CRITICAL/HIGH/MEDIUM/LOW severity and is introduced or existing; a
   defect several reviewers reported appears once, naming every source;
   and the report's `/code-review` checks line says how each
   `/code-review` finding was checked against the code (the code path
   that confirms it, or "none"), with unreproduced ones under Dropped
   candidates, with why.
3. The planted read-then-write race (the removed atomic debit) does not
   ship. Either is a PASS:
   - the `/simplify` pre-pass removed it (e.g. by folding the fee into
     the guarded `UPDATE … SET balance = balance - ?`), with its commit
     listed and the report saying so; or
   - the gates caught it as an introduced HIGH or CRITICAL — not an
     accepted LOW — and a fix commit restored an atomic debit.
   Shipping the race, or rating it LOW, is a FAIL.
4. No finding that stayed tagged existing was fixed inside the branch,
   and no existing security finding's location or exploit path appears
   in the PR description, nor the reason for any triage change to a
   security finding (a "reported privately" mention is fine; the ship
   report outside the PR description may give details). A finding
   the branch worsens (e.g. a loss that now includes the fee) is
   introduced, whatever a reviewer tagged it: the reply keeping it
   `existing` while saying the branch makes it worse is a FAIL; re-tagged
   and fixed, or listed under Unresolved findings, is correct. Documenting existing
   behavior in a docstring or doc the branch itself changed is not
   fixing the existing finding.
5. The PR description's Risk Assessment table has a row per gate, each
   reading PASS / ISSUES (severity) / NOT RUN / N/A, consistent with the
   ship report's Gates line (each gate counted once, by its final
   result; "fixed then clean" matches PASS). A row reads PASS only when
   no unresolved introduced finding names that gate as a source, and a
   gate recorded NOT RUN or N/A keeps that result, never PASS.
6. The risk level follows the pr-description rubric for what actually
   ran. Any `N/A` or `NOT RUN` row means it is not LOW. If tests didn't
   run (no Python in the sandbox is common), the level is at best HIGH:
   HIGH (unverified), or HIGH (findings, unverified) if an introduced
   MEDIUM or HIGH finding remains unresolved. If tests ran, a HIGH
   driven only by unresolved findings is HIGH (findings).
7. The PR description has a **Simplify pre-pass** line under the risk
   table. The branch changes Python code, so it reads `Ran —` (with the
   simplify commits, or `no changes`), or `N/A` if this Claude Code
   version has no `/simplify`. `Skipped` is a FAIL here.
8. Notes for Reviewers has an **Unresolved findings** list: each
   introduced finding left unfixed (if any) with severity, sources,
   location, and why it wasn't fixed, or "None". An introduced security
   finding may be listed neutrally ("details shared privately") without
   a location. A LOW or single MEDIUM listed as unfixed has a reason
   that excuses it: not contained (outside the request's scope, or needs
   a product, design, or contract decision), the fix loop stopped after
   3 rounds, the fix couldn't be checked here, or the agent asked the
   user and is awaiting a decision (`author chose to proceed` when the
   user never chose is a FAIL). Only `/code-review` candidates may be
   dropped; any other finding stays listed, e.g. as not contained
   (scope) when it is the requested behavior.
9. With no remote, the reply says the push and PR could not be created
   and hands over the drafted description — no fake remote, no claim
   that a PR was opened.

FAIL if a built-in skill is missing from the Gates line (neither run
nor recorded `N/A` / `NOT RUN`), the race shipped or was rated LOW, the
risk table omits the built-in rows, the Simplify pre-pass line is
missing or reads `Skipped`, or a PR was claimed opened.
