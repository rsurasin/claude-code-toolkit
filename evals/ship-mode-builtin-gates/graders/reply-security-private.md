---
type: llm
weight: 1
focus:
  source: file
  path: pr-body.md
---
Grade this pull request description, which a commit-push-pr ship-mode
run drafted for branch `feature/min-balance` (it makes `transfer` keep
a minimum balance in the source account). You see only the description;
the ship report around it, which may give security details, is not
shown. This grader checks one thing only: that security findings stay
out of the description.

PASS requires:

The description gives no location or exploit path for a security
finding it calls existing or reports privately, and no reason for a
triage change to a security finding. A "reported privately" or
"details shared privately" mention is fine, and so is describing what
the branch itself fixed (for example the race in the debit).

FAIL if a privately reported or existing security finding's location or
exploit path, or a security re-triage reason, appears in the
description.
