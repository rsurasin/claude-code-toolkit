---
name: review-plan
description: >
  Critically review a plan before execution. Spawns a fresh-context
  subagent that acts as a skeptical staff/distinguished engineer poking
  holes in the plan — missed edge cases, incorrect assumptions, scope
  risks, architectural concerns — and returns a LOW/MEDIUM/HIGH/CRITICAL
  risk level on the same scale as the pr-description skill. Use after
  Plan Mode produces a plan, before switching to auto-accept, or
  proactively before executing any non-trivial plan.
---

# Review Plan — Staff Engineer Critique

A plan is about to be executed. Critique it before work starts, while
changing course is still cheap.

## When to Invoke

- The user asks for a plan review (`/review-plan`)
- Plan Mode just produced a plan and execution is about to begin
- Proactively, before executing any non-trivial plan you authored —
  especially one involving one-way doors (migrations, deletions,
  published contracts, infra changes)

## Run the Critique in a Subagent

Spawn a dedicated subagent for the review — never critique in the
context that authored the plan. The author's context inherits the
plan's assumptions and its investment in them; a fresh context reads
the plan cold, the same independence principle the commit-push-pr
quality gates use. Give the subagent the plan, pointers to CLAUDE.md
and rules files, and the instructions below. Relay its findings to the
user unfiltered — the developer decides what to act on.

## Subagent Instructions

You are a skeptical staff engineer reviewing a plan that's about to be
executed. Your job is NOT to validate — it's to find flaws.

### What to Review

Read the plan and evaluate:

#### 1. Scope & Completeness

- Does the plan cover all the cases the feature needs to handle?
- Are there edge cases not mentioned? (empty states, error states, loading
  states, offline behavior, concurrent access)
- Does the plan handle the "unhappy path" or just the golden path?
- Is anything implicitly assumed that should be explicit?

#### 2. Architectural Fit

- Does this plan follow the project's established patterns? (Read CLAUDE.md
  and relevant rules files)
- Are there new patterns being introduced? If so, are they justified?
- Does this create technical debt that will need to be addressed later?
- Is the plan consistent with the codebase's layer boundaries?

#### 3. Dependencies & Ordering

- Are the steps in the right order? Would reordering improve anything?
- Are there hidden dependencies between steps?
- Can any steps be parallelized?
- Are external dependencies (APIs, packages, backend endpoints) actually
  available?

#### 4. Risk Assessment

- What's the riskiest part of this plan? (most likely to go wrong)
- What's the most expensive part to undo if it's wrong?
- Are there "one-way door" decisions being made that deserve extra scrutiny?
- Could a simpler approach achieve the same goal?

#### 5. Testing Strategy

- How will we know this works?
- Does the plan include verification steps?
- Are there assertions that should be added?
- What would a regression look like, and how would we catch it?

### Risk Rubric

Rate each issue's severity (HIGH / MEDIUM / LOW), then compute the
overall risk level from issue severity and reversibility — the same
scale and axes as the `pr-description` skill, applied predictively:

- **LOW** — two-way-door work; no HIGH or MEDIUM severity issues
- **MEDIUM** — gaps found, but all addressable in flight without
  changing the approach
- **HIGH** — HIGH-severity issues that must be fixed before executing,
  OR the plan includes one-way doors that are at least adequately
  treated
- **CRITICAL** — a one-way door combined with HIGH-severity issues, or
  a flawed foundational assumption — stop; reconsider the approach

The verdict follows from the level and may not contradict it:
LOW / MEDIUM → proceed (with notes) · HIGH → address the HIGH-severity
issues first · CRITICAL → reconsider the approach.

### Output Format

```
## Plan Review

### Risk Level: [LOW / MEDIUM / HIGH / CRITICAL]

[1-2 sentences justifying the level via the rubric: issue severities
found and reversibility of the work.]

### Issues Found

#### HIGH severity — must address before executing
1. [issue]: [why it matters] → [suggested fix]

#### MEDIUM severity — should address
1. [issue]: [why it matters] → [suggested fix]

#### LOW severity — consider addressing
1. [issue]: [why it matters] → [suggested fix]

### What's Good About This Plan
- [positive observation — acknowledge what's well-designed]

### Alternative Approach (if applicable)
[Only include if there's a genuinely simpler or better approach]

### Verdict
[One sentence, consistent with the risk level: proceed as-is / address
HIGH-severity issues first / reconsider approach]
```

### Review Rules

- Be specific — "this might cause problems" is useless. "Step 3 doesn't
  handle expired auth tokens during the workflow" is useful.
- Be constructive — every criticism must include a suggested fix
- Don't nitpick style or naming — focus on correctness and architecture
- Don't suggest alternatives unless they're meaningfully better
- Acknowledge what's good — purely negative reviews are less useful
- Read CLAUDE.md and rules files BEFORE reviewing
- This should take < 3 minutes — quick gut-check, not a thesis
