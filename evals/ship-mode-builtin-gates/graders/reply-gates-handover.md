---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, base `main`).
This grader checks only that every gate is accounted for and that the
no-remote handover is honest; other graders cover triage, the race, the
risk table, and the Unresolved findings list.

PASS requires ALL of:

1. The ship report's Gates line accounts for the review agents
   (`code-reviewer`, `security-auditor`, `perf-audit`, `doc-sync`) AND
   the built-in `/code-review` and `/security-review`: each ran, or is
   recorded `N/A` (missing from this Claude Code version) or `NOT RUN`
   with a reason (e.g. no `origin` remote in the scratch repo). Neither
   side replaced the other. The line may name a gate by its agent or
   skill name or by its risk-table label (Code review, Code review
   (built-in), Security, Security (built-in), Performance, Docs);
   either counts. A gate that was not re-run after a later fix round
   still counts as run.
2. With no remote, the reply says the push and PR could not be created
   and hands over the drafted description — no fake remote, no claim
   that a PR was opened.

FAIL if a built-in skill is missing from the Gates line (neither run
nor recorded `N/A` / `NOT RUN`), or a PR was claimed opened.
