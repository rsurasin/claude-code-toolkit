# Checkpoint Mode

Part of the `commit-push-pr` skill; read this file in checkpoint mode.
Step numbers refer to `ship-mode.md`; the Rules in `SKILL.md` apply.

A plan estimated over ship mode's soft limit (400 lines or 20 files) is
split into phases (`dev-tasks`). A checkpoint reviews each phase when it
ends, while it is small enough to review well. It doesn't replace ship
mode: the ship pass still runs every gate and hard stop over the whole
branch, and reviews the checkpoint's fix commits.

## When it runs

Right after commit mode commits the last task of a phase recorded in the
active dev-tasks `tasks.md`, or of a phase plan the user gave in the
conversation, or when the user asks for a checkpoint. Never mid-phase.

It reviews `<phase start>..HEAD`, from the phase's recorded start
commit. With no start recorded, say so in the report and review
nothing, since a guessed range reviews the wrong code.

**It never blocks, never asks, and never hides.** Attended or not, it
asks nothing and opens no PR: where a step below would ask, it puts the
question in the report instead.

## Steps

1. **Review.** Run these Step 2 gates over the phase range, in place of
   `<base>...HEAD`, unattended and report-only as there: Tests (item 1),
   `code-reviewer`, `security-auditor`, and `/code-review` at `high`
   with the range as its target and item 7's range check. Not
   `/security-review` (it reviews only the whole branch against
   `origin/HEAD`), `perf-audit`, `doc-sync`, runtime verification or
   `/simplify`: those stay in the ship pass.
2. **Triage.** Step 2b, with the phase range, the phase start as base
   ref, and as scope only the phase's tasks written out as the request
   (no issue, no plan), so findings about later phases come back not
   contained and are recorded, not fixed. Existing findings go in the
   report only. If triage fails twice, report it and stop.
3. **Fix.** `fix-loop.md`, one round only: items 1–4 of a round, over
   every finding marked must fix or contained, LOW included. Label the
   records `AUDIT (checkpoint, <phase>)` and `checkpoint`. Then re-run
   Tests and revert any fix commit that breaks one. Tick a finding's
   `## Review Fixes` line once its fix is committed and Tests pass. No
   second round, no cleanup pass, no closing review, no re-run of the
   reviewers and no re-triage. The scope freeze of Step 2b item 1 ends
   here.
4. **Record** the reviewed head as the phase's `Checkpoint:` in
   `tasks.md` (when a task folder exists), and push as commit mode does.

## Report

```
## Checkpoint

**Phase:** <what the phase delivers> (<start>..<head>)
**Reviewed by:** <gates that ran>; not run: <names (why), or none>
**Fixed:** <n> findings (<commits>), not re-reviewed until ship mode
**Still open:** <severity, sources, file:line, reason — or none>
**Needs your attention:** <every introduced CRITICAL or HIGH still open, and every question, or none>
```

An introduced CRITICAL or HIGH still open is never dropped: it stays in
`tasks.md` and on the Needs your attention line, and ship mode's gates
meet it again at PR time.
