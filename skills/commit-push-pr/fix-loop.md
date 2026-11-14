# Fix Loop

Part of the `commit-push-pr` skill; read this file in ship mode Step 3,
after `finding-triage` returned its list (Step 2b of `ship-mode.md`).
Step numbers below refer to `ship-mode.md`; the Rules in `SKILL.md`
apply here too.

The loop is bounded: **at most two fix rounds.** Round 1 fixes what the
first triage found. Round 2 repairs round 1 and fixes any serious
finding a re-run raised late. Minor findings left after that are listed
under Unresolved findings; every fix can raise new ones, so more rounds
do not converge. The bound never lets a serious finding through: **a PR
never opens with an introduced CRITICAL or HIGH finding unfixed** unless
the user said to proceed.

## When round 1 runs

Round 1 runs when `finding-triage` reports **Fix loop: REQUIRED** (any
introduced CRITICAL or HIGH, or two or more introduced MEDIUM), or when
it called a single introduced MEDIUM **contained**. LOW findings never
start a round: when nothing else starts one, no round runs and each LOW
is listed with `LOW: no fix round for it`.

## A round

1. Report the findings to the user
2. Append them to the active dev-tasks context file, if one exists
3. Fix this round's findings (round 1: every finding marked **must
   fix** or **contained**; round 2: see below). Each fix is the
   smallest change that resolves its finding and passes
   `finding-triage`'s no-design-or-contract-change test (its step 6).
   When only a design or contract change would resolve a must-fix
   finding, don't make it: ask the user, and list the finding as
   `awaiting author decision` until they answer. Never delete, skip,
   or weaken a test to make a gate pass; a test you believe is wrong
   is a question for the user
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
   previous list, the commit the round started from, the fix commits,
   the re-run gates' reports, and any contained fix you reverted. It
   carries forward every earlier finding the re-run didn't cover, marks
   every new finding `fix-caused` or `late`, and grades your fixes as
   cold as the original code. Check its list as in Step 2b item 2; a
   failed re-triage follows Step 2b's retry rule.

The contained call is the agent's, not yours: fix every contained
finding, and never mark one not contained yourself (Step 2b item 2). If
a contained fix turns out bigger once started (it needs a design or
contract change, or spreads well beyond the requested behavior), revert
that fix and pass the agent the revert commit and what the fix needed
at the re-triage: what it needed, not a call. If it says contained
again and you still can't make the fix here, put both positions to the
user and list the finding as `awaiting author decision`.

## After round 1

The re-triage marks each finding it had not listed before:

- **`fix-caused`** — in lines a fix commit added or changed.
- **`late`** — anywhere else: a re-run raised it on code no fix touched.

Then:

- A **late CRITICAL or HIGH** is a real defect the first pass missed:
  it is must fix, in round 2.
- A **late MEDIUM or LOW** is not fixed in this loop. List it with
  `late: raised by a re-run on code the fixes didn't touch`.
- **Round 2 runs only** when a must-fix finding is still unresolved, a
  late finding is CRITICAL or HIGH, or a fix-caused finding is MEDIUM
  or higher. It fixes those findings and nothing else (items 3–5
  again). LOW findings never start or extend a round: a fix-caused LOW
  is listed with `LOW: no fix round for it`.
- **After round 2's re-run and re-triage, stop.** Never start a third
  round unless the user asks for one in this conversation. A MEDIUM or
  LOW still open is listed with `stopped after 2 fix rounds`. **An
  introduced CRITICAL or HIGH still open, or failing tests, is a hard
  stop** (Step 3): no ready-to-merge PR opens. Report it under
  **Decisions needed** with what you tried, and let the user choose
  between another round, a different fix, or proceeding.

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
- `late: raised by a re-run on code the fixes didn't touch`
- `LOW: no fix round for it`
- `stopped after 2 fix rounds`

That list is the final triage's Introduced section, one entry per
finding: a finding leaves it only when a re-triage lists it under
Resolved (a contained finding whose fix reads "none needed" stays), so
a review gate's risk-table row reads ISSUES exactly when an Unresolved
entry names that gate. Clean gates proceed directly. `NOT RUN` and
`N/A` gaps don't block — they are reflected honestly in the risk level
instead. Existing findings never block, except an existing CRITICAL
security finding (Step 3 hard stop); Step 2b already routed them.
