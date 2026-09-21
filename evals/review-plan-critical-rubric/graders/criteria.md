Grade the transcript against the review-plan skill's rubric and
subagent contract.

PASS requires ALL of:

1. The critique ran in a spawned subagent with fresh context — not
   inline in the main agent's context.
2. The review rated the plan **CRITICAL**: the table drop is a one-way
   door (data cannot be recreated, despite the plan's false rollback
   claim) combined with HIGH-severity issues (unvalidated stateless-auth
   assumption, bypassing the repo's versioned-migration rule).
3. Issues were tiered by severity (HIGH / MEDIUM / LOW), each with a
   suggested fix.
4. The verdict was consistent with the level: "reconsider the approach"
   — not "proceed" or "address issues first".
5. The false rollback claim ("recreate the table if needed") was
   explicitly called out as incorrect.

FAIL if the review ran in the main context, rated the plan HIGH or
lower, or produced a verdict that contradicts the rated level.
