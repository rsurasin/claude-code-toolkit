# Fix Loop

Part of the `commit-push-pr` skill; read this file in ship mode Step 3,
after `finding-triage` returned its list (Step 2b of `ship-mode.md`).
Step numbers below refer to `ship-mode.md`; the Rules in `SKILL.md`
apply here too.

**Fix-before-PR loop — findings do not ship:**

If `finding-triage` reports **Fix loop: REQUIRED** (any introduced
CRITICAL or HIGH finding, or two or more introduced MEDIUM findings):

1. Report the findings to the user
2. Append them to the active dev-tasks context file, if one exists
3. Fix every finding it marked **must fix** — plus, in the same round,
   every finding it marked **contained** (see below). Each fix, must
   fix included, is the smallest change that resolves its finding and
   passes `finding-triage`'s no-design-or-contract-change test (its
   step 6). When only a design or contract change would resolve a
   must-fix finding, don't make it: ask the user, and list the finding
   as `awaiting author decision` until they answer
4. Commit the fixes atomically (Commit Mode Steps 3–4, e.g.
   `fix(<scope>): <subject>`) — the gates review `<base>...HEAD`, so an
   uncommitted fix is invisible to them and missing from the PR
5. Re-run Tests and runtime verification, plus only the affected review
   gates — the sources named on the findings you fixed. Each re-run
   still passes `<base>...HEAD` as its range and reference point (so an
   incomplete fix stays `introduced`, and `/code-review`'s range check
   holds); the fix commits' files and the flagged locations are its
   *focus*, not its range (`security-auditor` skips DAST and
   full-codebase tooling on a re-run unless the finding came from that
   tooling). Then re-triage: dispatch `finding-triage` again with
   everything Step 2b item 1 gave it (range, base ref, scope), plus its
   previous list, the fix commits, the re-run gates' reports, and any
   contained fix you reverted (below). It carries forward every earlier
   finding the re-run didn't cover, so each row reflects the whole
   branch, and grades your fixes as cold as the original code. Check
   its list as in Step 2b item 2; a failed re-triage follows Step 2b's
   retry rule.
6. Repeat until it reports the fix loop NOT REQUIRED, or the user says to
   proceed anyway. After three fix rounds, stop and ask the user whether
   to continue, listing the remaining findings.

**Below-threshold findings — fix the contained ones too:**

Every other introduced finding (any number of LOW, or a single MEDIUM)
is fixed in this PR when `finding-triage` calls it **contained**: in
the PR's scope (measured against the issue, the task plan, and the
request, not the diff), no design or contract change, and small enough
not to warrant its own issue. Its agent definition has the full tests.
The call is the agent's, not yours: fix every contained finding, and
never mark one not contained yourself (Step 2b item 2). If a contained
fix turns out bigger once started (it needs a design or contract change,
or spreads well beyond the requested behavior), revert that fix and pass
the agent the revert commit and what you found at the next re-triage.
This is one of the two re-dispatch inputs (Step 2b item 1): what the
fix needed, not a call. It re-runs the three tests against the code
and returns `contained` or `not contained`. If it says contained again
and you still can't make the fix here, put both positions to the user
(Step 2b item 2); the finding never ships without a call or a user
decision.

Fold contained fixes into the fix loop's current round, so they cost no
extra gate re-run. If the threshold was never met, run one round of the
loop (items 3–5) for the contained fixes alone. A **not contained**
finding stays unfixed with the agent's reason — which test it failed
and why, citing the issue, never a dev-tasks file (e.g. "not contained:
scope — #31 covers reset by email link, not MFA") — and you offer to
file it as an issue with that reason as the draft (`gh issue create`
once the user approves). Every introduced
finding still unfixed when the PR opens — not contained, awaiting the
user's decision, the user said to proceed, or the loop stopped after
three rounds — goes in the PR's **Unresolved findings** list (Step 5)
with its reason, and the PR proceeds. That list is the final triage's
Introduced section, one entry per finding: a finding leaves it only
when a re-triage lists it under Resolved (a contained finding whose fix
reads "none needed" stays), so a review gate's risk-table row reads
ISSUES exactly when an Unresolved entry names that gate. Clean gates
proceed directly. `NOT RUN` and `N/A` gaps don't block — they are
reflected honestly in the risk level instead.
Existing findings never block, except an existing CRITICAL security finding (hard stop above);
Step 2b already routed them.
