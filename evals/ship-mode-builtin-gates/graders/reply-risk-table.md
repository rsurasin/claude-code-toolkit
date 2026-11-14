---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, base `main`,
a Python change). This grader checks only the PR description's Risk
Assessment: the table, the risk level, and the Simplify pre-pass line,
against the pr-description risk rubric. Another grader checks that the
table has the two built-in rows by name.

PASS requires ALL of:

1. The Risk Assessment table has a row per gate, each reading PASS /
   ISSUES (severity) / NOT RUN / N/A, consistent with the ship report's
   Gates line (each gate counted once, by its final result; "fixed then
   clean" matches PASS). A row reads PASS only when no unresolved
   introduced finding names that gate as a source, and a gate recorded
   NOT RUN or N/A keeps that result, never PASS.
2. The risk level follows the pr-description rubric for what actually
   ran. Any `N/A` or `NOT RUN` row means it is not LOW. If tests didn't
   run (no Python in the sandbox is common), the level is at best HIGH:
   HIGH (unverified), or HIGH (findings, unverified) if an introduced
   MEDIUM or HIGH finding remains unresolved. If tests ran, a HIGH
   driven only by unresolved findings is HIGH (findings).
3. The PR description has a **Simplify pre-pass** line under the risk
   table. The branch changes Python code, so it reads `Ran —` (with the
   simplify commits, or `no changes`), or `N/A` if this Claude Code
   version has no `/simplify`. `Skipped` is a FAIL here.

FAIL if a row's result disagrees with the Gates line, the level is
LOW with any `NOT RUN` or `N/A` row, or the Simplify pre-pass line is
missing or reads `Skipped`.
