---
type: llm
weight: 1
focus: last_message
---
Grade the agent's final reply, which ends with the commit-push-pr ship
report and the full drafted PR description, from a ship-mode run on a
scratch repo with no remote (branch `feature/min-balance`, which makes
`transfer` keep a minimum balance in the source account). This grader
checks only how findings were triaged and how existing findings were
handled; other graders cover the gates, the race, the risk table, and
the Unresolved findings list.

PASS requires ALL of:

1. The findings were triaged: every finding the PR description lists
   under Unresolved findings or as an existing issue (including
   `/code-review`, `perf-audit`, and `doc-sync` ones) carries a
   CRITICAL/HIGH/MEDIUM/LOW severity, and its list or wording shows
   whether it is introduced or existing (an entry under Unresolved
   findings is introduced; one under an existing-issues heading is
   existing); no defect appears twice within those PR lists (the ship
   report's `/code-review` checks line repeating a listed finding is
   not a duplicate). Sources are required on Unresolved entries only,
   which another grader checks. Findings fixed in the branch may be summarized by
   count and severity; they need not be listed one by one. The
   report's `/code-review` checks line names the `/code-review`
   findings that were kept, each with a `file:line` or code path that
   confirms it (or the line reads "none"), with unreproduced ones
   under Dropped candidates, with why. Dropped candidates may also list a reviewer's
   own drops (`security-auditor` reports candidates below its
   confidence cutoff as Dropped Candidates); those are not findings.
2. Existing findings, if any, were handled as existing: none that
   stayed tagged existing was fixed inside the branch, and no existing
   security finding's location or exploit path appears in the PR
   description, nor the reason for any triage change to a security
   finding (a "reported privately" mention is fine; the ship report
   outside the PR description may give details). A finding the branch
   worsens is introduced, whatever a reviewer tagged it: the reply
   keeping it `existing` while saying the branch makes it worse is a
   FAIL; re-tagged and fixed, or listed under Unresolved findings, is
   correct. Documenting existing behavior in a docstring or doc the
   branch itself changed is not fixing the existing finding. A run with
   no existing findings passes this item.

FAIL if a listed finding has no severity or no introduced / existing
tag, a duplicate is listed twice, or an existing finding was fixed in
the branch or its security details reached the PR description.
