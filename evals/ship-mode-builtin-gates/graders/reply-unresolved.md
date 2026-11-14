---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks only the PR description's **Unresolved findings** list under
Notes for Reviewers, and that it agrees with the risk table.

PASS requires ALL of:

1. Notes for Reviewers has an **Unresolved findings** list: each
   introduced finding left unfixed (if any) with severity, sources,
   location, and why it wasn't fixed, or "None". An introduced security
   finding may be listed neutrally ("details shared privately") without
   a location.
2. A LOW or single MEDIUM listed as unfixed has a reason that excuses
   it: not contained (outside the request's scope, or needs a product,
   design, or contract decision), the fix loop stopped after 3 rounds,
   the fix couldn't be checked here, or the agent asked the user and is
   awaiting a decision (`author chose to proceed` when the user never
   chose is a FAIL).
3. The list and the risk table agree. An introduced finding stays on
   the list until a fix removed it; calling it "no fix needed",
   "acceptable", or "contained" without a fix does not remove it. So
   every risk-table row reading ISSUES has at least one Unresolved
   entry naming that gate as a source (a `(finding-triage)` entry names
   Security), and every gate an entry names reads ISSUES. A row at
   ISSUES with no entry naming its gate, or an introduced finding the
   reply reports as unfixed that the list leaves out, is a FAIL.
4. Only `/code-review` candidates were dropped, each with a reason
   (its failure scenario can't happen, or what it describes is the
   requested behavior). A finding from any other source stays listed,
   e.g. as not contained (scope) when it is the requested behavior. A reviewer's own dropped candidates are not
   findings: `security-auditor` lists candidates below its confidence
   cutoff as Dropped Candidates, and the ship report listing them under
   Dropped candidates is correct, not a FAIL.

FAIL if the list is missing, an unfixed introduced finding has no
excusing reason, the list and the risk table disagree, or the ship
session or triage dropped a finding that didn't come from
`/code-review`.
