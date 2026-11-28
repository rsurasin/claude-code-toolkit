# Fix Loop

Part of the `commit-push-pr` skill; read this file in ship mode Step 3,
after `finding-triage` returned its list (Step 2b of `ship-mode.md`).
Step numbers below refer to `ship-mode.md`; the Rules in `SKILL.md`
apply here too.

Every introduced finding is fixed before the PR opens, LOW ones
included: small findings left behind pile up as debt. The work is
bounded so it converges: **at most two fix rounds, then one cleanup
pass.** A round re-runs the reviewers, which is what finds new
problems; the cleanup pass fixes the LOW findings that are left, and a
closing review then reads only the cleanup commits and can only revert
them, never start more fixing, so the work ends there. The bound never
lets a serious finding through: **no mergeable PR opens with an
introduced CRITICAL or HIGH finding unfixed**. If the user says to
proceed, the PR opens as a draft; an unattended run opens a
`[BLOCKED]` draft (Step 3).

## Fix rounds (MEDIUM and above)

A round runs while an introduced finding of MEDIUM or higher is open
and fixable: one marked **must fix**, or a single MEDIUM marked
**contained**. A round also runs while a test this branch broke is
failing (Step 3 tells those from tests already failing on the base):
treat it like a must-fix finding and fix the code, never the test. In
each round:

1. Report the findings to the user
2. Record them in the active dev-tasks folder, if one exists, so the
   audit trail shows what review found and what was done about it:
   an `AUDIT (ship gates, round <n>)` entry at the end of `context.md`
   listing each finding with its severity and sources, and one line
   per finding under `## Review Fixes` in `tasks.md`. Tick a line and
   add the fix commit once a re-triage lists the finding under
   Resolved; a finding that ships unfixed keeps an open line with its
   Unresolved reason. Add nothing else to `tasks.md` or `plan.md`
3. Fix every open finding marked **must fix** or **contained**, LOW
   ones included (they cost no extra re-run). Each fix is the smallest
   change that resolves its finding and passes `finding-triage`'s
   no-design-or-contract-change test (its step 6). When only a design
   or contract change would resolve a must-fix finding, don't make it:
   ask the user, and list the finding as `awaiting author decision`
   until they answer (for a CRITICAL or HIGH, that is a hard stop too).
   Never delete, skip, or weaken a test to make a gate pass; a test you
   believe is wrong is a question for the user. In a harness file (the
   list under Step 5's Needs human review line), fix only a factual
   claim, such as a doc-sync `(harness)` finding, and never a rule,
   decision, prohibition or workflow text: a finding that needs one
   changed is `awaiting author decision`. Edit the file's real path
   (`readlink -f`), never through a link
4. Commit the fixes atomically (Commit Mode Steps 3–4, e.g.
   `fix(<scope>): <subject>`) — the gates review `<base>...HEAD`, so an
   uncommitted fix is invisible to them and missing from the PR
5. Re-run Tests and runtime verification, plus the affected review
   gates — the sources named on the findings you fixed — and always
   `code-reviewer` and `security-auditor`, whether or not they raised
   one of those findings, so every fix commit gets a code read and a
   security read (a fix for a non-security finding can still open a
   security hole). A gate that is neither is not re-run. Each re-run
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
still open, or a test the branch broke still failing, is a hard stop**
(Step 3): no PR opens
for merging. Report it under **Decisions needed** with what you tried,
and let the user choose between another round, a different fix, or
proceeding (which opens the PR as a draft: Step 3).

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

1. Record them as in a round's item 2 (`AUDIT (ship gates, cleanup)`),
   fix every one of them, each with the smallest change (in a harness
   file, facts only, as in a round's item 3), and commit atomically (e.g. `refactor(<scope>): …`, `test(<scope>): …`,
   `docs(<scope>): …`)
2. Re-run Tests and runtime verification. If a test fails, revert the
   cleanup commit that broke it
3. **Closing review.** No commit ships unreviewed: dispatch
   `code-reviewer` and `security-auditor` once more, each told to
   review only what the cleanup commits changed
   (`git diff <commit before the cleanup pass>..HEAD`) and to report
   only defects those changes introduce. Don't re-run the other gates
4. Re-triage once more, with the cleanup commits and the closing
   review's two reports. `finding-triage` confirms each fix in the
   code: a finding it lists under Resolved is done; one it doesn't is
   listed with `cleanup fix not confirmed`
5. **The closing review can only take a cleanup commit back.** If it
   or the re-triage finds a defect, of any severity, that a cleanup
   commit introduced, revert that commit and re-run Tests. Never fix
   what the closing review found: a revert restores code the gates
   already reviewed, so it needs no further review or re-triage, and
   nothing new is left to chase. A finding the closing review raises
   that the re-triage ties to no cleanup commit (a defect in code the
   cleanup pass didn't change) is not reverted and not fixed either:
   it stays on the triage list as graded. A LOW or MEDIUM ships listed
   with `found by the closing review`; a CRITICAL or HIGH is a hard
   stop at once (Step 3), with no further round unless the user asks
   for one
6. **Bring the list in line with the reverts.** Step 4's triage read
   the code before any revert, so for each reverted cleanup commit,
   and only for those: take off the list every finding the triage
   attributed to that commit (the revert removed it, so it is not a
   finding and never a hard stop), and put back the LOW the commit was
   fixing, with `cleanup fix not confirmed`, even if the triage listed
   it under Resolved. This is bookkeeping for a revert, not a triage
   call: change nothing else on the list. Step 4's Gate verdicts
   table predates the reverts too, so after a revert don't copy it:
   work out each reviewing gate's row from the corrected list (PASS
   when no entry names that gate as a source, otherwise ISSUES at the
   highest severity among the entries that do; a `(finding-triage)`
   entry names Security). The risk level and the hard-stop check use
   those rows. A gate recorded `NOT RUN` or `N/A` stays as recorded

There is no second cleanup pass and no second closing review.

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
  conversation; the PR then opens as a draft
- `stopped after 2 fix rounds` — a finding the rounds didn't resolve
- `not attempted: run stopped at <hard stop>` — the run hit a hard
  stop before the loop could work on the finding (a test already
  failing on the base)
- `not triaged: triage failed twice (<n> raw findings, in the ship
  report)` — triage failed twice (`ship-mode.md` Step 2b), so nothing
  graded the findings. Unlike the other reasons, this is one line for
  the whole list, not one per finding: it gives the count and names no
  finding, severity, source or location, since nothing has sorted out
  which are security findings
- `cleanup fix not confirmed` — a LOW whose cleanup fix was reverted or
  not confirmed
- `found by the closing review` — a finding the closing review raised
  in code no cleanup commit changed (cleanup step 5): the loop fixes
  nothing after the cleanup pass
- `no change recommended by triage — <its reason>` — a finding whose
  Fix line in the triage list says none is needed

An introduced CRITICAL or HIGH is never listed in a ready PR. In ship
mode it appears only in a draft: with `author chose to proceed` when
the user proceeded, or in a `[BLOCKED]` draft with whichever of
`stopped after 2 fix rounds`, `awaiting author decision`,
`not attempted` or `found by the closing review` describes it. A draft opened after a failed triage
carries only the `not triaged` line: with no triage list, no finding
has a severity or an introduced tag to list it by.

Release mode adds one reason of its own, `release shipped without a
fix PR` (`release-mode.md`); ship mode never uses it.

Except after a failed triage, which has no list and gets the single
`not triaged` line, that list is the final triage's Introduced
section, after the cleanup
pass's step 6 if a cleanup commit was reverted, one entry per
finding: a finding leaves it only when a re-triage lists it under
Resolved (a contained finding whose fix reads "none needed" stays), so
a review gate's risk-table row reads ISSUES exactly when an Unresolved
entry names that gate. Clean gates proceed directly. `NOT RUN` and
`N/A` gaps don't block — they are reflected honestly in the risk level
instead. Existing findings never block; Step 2b already routed them
(an existing CRITICAL security finding is also flagged on the Needs
human review line, in neutral words: Step 3).
