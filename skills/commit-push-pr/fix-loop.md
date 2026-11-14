# Fix Loop

Part of the `commit-push-pr` skill; read this file in ship mode Step 3,
after `finding-triage` returned its list (Step 2b of `ship-mode.md`).
Step numbers below refer to `ship-mode.md`; the Rules in `SKILL.md`
apply here too.

Every introduced finding is fixed before the PR opens, LOW ones
included: small findings left behind pile up as debt. The work is
bounded so it converges: **at most two fix rounds, then one cleanup
pass.** A round re-runs the reviewers, which is what finds new
problems; the cleanup pass fixes the LOW findings that are left without
re-running them, so nothing new is raised after it. The bound never
lets a serious finding through: **a PR never opens with an introduced
CRITICAL or HIGH finding unfixed** unless the user said to proceed.

## Fix rounds (MEDIUM and above)

A round runs while an introduced finding of MEDIUM or higher is open
and fixable: one marked **must fix**, or a single MEDIUM marked
**contained**. In each round:

1. Report the findings to the user
2. Append them to the active dev-tasks context file, if one exists
3. Fix every open finding marked **must fix** or **contained**, LOW
   ones included (they cost no extra re-run). Each fix is the smallest
   change that resolves its finding and passes `finding-triage`'s
   no-design-or-contract-change test (its step 6). When only a design
   or contract change would resolve a must-fix finding, don't make it:
   ask the user, and list the finding as `awaiting author decision`
   until they answer (for a CRITICAL or HIGH, that is a hard stop too).
   Never delete, skip, or weaken a test to make a gate pass; a test you
   believe is wrong is a question for the user
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
   contained fix you reverted. It carries forward every earlier finding
   the re-run didn't cover, and grades your fixes as cold as the
   original code. Check its list as in Step 2b item 2; a failed
   re-triage follows Step 2b's retry rule.

**After the second round, stop.** Never start a third round unless the
user asks for one in this conversation. A MEDIUM still open is listed
with `stopped after 2 fix rounds`. **An introduced CRITICAL or HIGH
still open, or failing tests, is a hard stop** (Step 3): no PR opens
for merging. Report it under **Decisions needed** with what you tried,
and let the user choose between another round, a different fix, or
proceeding.

The contained call is the agent's, not yours: fix every contained
finding, and never mark one not contained yourself (Step 2b item 2). If
a contained fix turns out bigger once started (it needs a design or
contract change, or spreads well beyond the requested behavior), revert
that fix and pass the agent the revert commit and what the fix needed
at the next re-triage: what it needed, not a call. If it says contained
again and you still can't make the fix here, put both positions to the
user and list the finding as `awaiting author decision`.

## Cleanup pass (LOW)

Run this once, after the last round, or straight away when no round
ran, if any introduced LOW marked **contained** is still open:

1. Fix every one of them, each with the smallest change, and commit
   atomically (e.g. `refactor(<scope>): …`, `test(<scope>): …`,
   `docs(<scope>): …`)
2. Re-run Tests and runtime verification. Don't re-run the review
   gates: that is what keeps this pass from raising new findings. If a
   test fails, revert the cleanup commit that broke it
3. Re-triage once more, with the cleanup commits and no new gate
   reports. `finding-triage` confirms each fix in the code: a finding
   it lists under Resolved is done; one it doesn't is listed with
   `cleanup fix not confirmed`. If it grades a cleanup commit as
   introducing a MEDIUM or higher, revert that commit and list its
   finding the same way

There is no second cleanup pass.

## What ships unfixed

Every introduced finding still unfixed when the PR opens goes in the
PR's **Unresolved findings** list (Step 5) with one of these reasons,
and the PR proceeds:

- `not contained: scope|contract|size — <reason>` — the agent's reason,
  citing the issue, never a dev-tasks file (e.g. "not contained: scope —
  #31 covers reset by email link, not MFA"). Offer to file it as an
  issue with that reason as the draft (`gh issue create` once the user
  approves)
- `awaiting author decision` — you asked and have no answer yet
- `author chose to proceed` — only when the user said so in this
  conversation; the only reason a CRITICAL or HIGH may carry
- `stopped after 2 fix rounds` — a MEDIUM the rounds didn't resolve
- `cleanup fix not confirmed` — a LOW whose cleanup fix was reverted or
  not confirmed
- `no change recommended by triage — <its reason>` — a finding whose
  Fix line in the triage list says none is needed

That list is the final triage's Introduced section, one entry per
finding: a finding leaves it only when a re-triage lists it under
Resolved (a contained finding whose fix reads "none needed" stays), so
a review gate's risk-table row reads ISSUES exactly when an Unresolved
entry names that gate. Clean gates proceed directly. `NOT RUN` and
`N/A` gaps don't block — they are reflected honestly in the risk level
instead. Existing findings never block, except an existing CRITICAL
security finding (Step 3 hard stop); Step 2b already routed them.
