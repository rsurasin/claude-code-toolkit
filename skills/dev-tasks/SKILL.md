---
name: dev-tasks
description: >
  Maintain task-specific context files (plan, context, tasks) that persist
  across Claude Code sessions. Use when starting a non-trivial feature, when
  resuming work from a previous session, or when asked to create or update
  development task documentation. Bridges the "daily amnesia" problem.
---

# Dev Tasks Skill

Maintain structured task documentation that survives across Claude Code
sessions. Every non-trivial task gets a context folder. This is how you
prevent re-explaining the same constraints every session.

## When to Create Dev Tasks

- Starting any feature that will take more than one session
- When the user says "let's plan" or "I want to work on X"
- When resuming a task that already has dev task files
- When the user references a task by name

## Directory Structure

```
.claude/dev/[task-name]/
├── plan.md       # The accepted implementation plan
├── context.md    # Key files, decisions, constraints, gotchas
└── tasks.md      # Checklist of work items with status & log of actions performed under tasks as subbullets
```

Use `.claude/dev/` at the project root. Create it if it doesn't exist.
Use kebab-case for task names: `.claude/dev/search-feature/`, `.claude/dev/auth-refresh/`.

## File Formats

### plan.md

```markdown
# [Task Name] — Plan

## Goal
[One sentence: what this achieves for the user]

## Approach
[Numbered steps of the implementation strategy]

## Key Decisions
- [Decision 1]: [rationale]
- [Decision 2]: [rationale]

## Files to Modify
- [path]: [what changes]

## Files to Create
- [path]: [purpose]

## Dependencies / Blockers
- [any prerequisites or external dependencies]

## Open Questions
- [anything unresolved that needs clarification]
```

### context.md

```markdown
# [Task Name] — Context

## Relevant Code
- `[path]`: [what it does and why it matters for this task]

## Constraints
- [constraint 1]: [why]
- [constraint 2]: [why]

## Patterns to Follow
- [reference existing code that demonstrates the pattern]

## Gotchas Discovered
- [gotcha 1]: [what happened and how to avoid it]

## Session Log
- [date]: [what was accomplished, what's left]
```

### tasks.md

```markdown
# [Task Name] — Tasks

## In Progress
- [ ] [task description]
    - [x] Step 1. [Exact changes made]
    - [ ] Step 2. [Exact changes made]

## Done
- [x] [date] [task description] — [any notes]
    - [x] Step 1. [Exact changes made]
    - [x] Step 2. [Exact changes made]
    - [x] Step 3. [Exact changes made]

## Blocked
- [ ] [task description] — blocked by: [reason]

## Discovered During Implementation
- [ ] [new task found while working]
```

## Session Start Behavior

When beginning work on a task that has existing dev task files:

1. Read all three files in the task folder
2. Summarize current status: what's done, what's in progress, what's next
3. Check if the plan is still accurate given any code changes since last session
4. If plan has drifted from code, flag the discrepancies
5. Continue from where the last session left off

## Session End Behavior

Before ending a session or switching tasks:

1. Update `tasks.md` with completed and remaining items
2. Add any gotchas discovered to `context.md`
3. Add a session log entry to `context.md` with date and summary
4. If the plan changed during the session, update `plan.md`

## Rules

- Dev task files are living documents — update them as you learn, not just at start
- Keep `plan.md` concise — if it exceeds 200 lines, the task should be split
- `context.md` is the place for things you'd forget between sessions
- `tasks.md` should be copy-pasteable as a PR checklist
- Never delete session log entries — they're a breadcrumb trail
- If the user says "catch me up" or "what were we working on", read the dev
  task files and summarize
- Add `.claude/dev/` to `.gitignore` — these are working notes, not project artifacts
