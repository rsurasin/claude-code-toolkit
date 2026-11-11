---
type: llm
weight: 1
focus: { source: file, path: .claude/dev/features/2026-10-04-gh-31-password-reset/tasks.md }
---

Grade this `tasks.md` against the dev-tasks status contract. The setup
ended it with `Status: PR open (#41)`; the user then reported that PR #41
merged into `dev` as commit `9f3c2e1`.

PASS requires ALL of:

1. The last non-blank line is exactly
   `Status: merged to dev (9f3c2e1), not released`.
2. Nothing follows the status line. A `## Release Steps` section, if the
   agent added one, sits above it.

FAIL if the status line uses other wording ("merged", "done",
"released", "awaiting release"), still reads `PR open (#41)`, or is not
the last line.
