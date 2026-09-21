---
name: dev-tasks
description: >
  Maintain task-specific context files (plan, context, tasks) under
  .claude/dev/ that persist across Claude Code sessions. Use when starting a
  non-trivial feature, when resuming work from a previous session, or when
  asked to create or update development task documentation. Bridges the "daily
  amnesia" problem. Repo-local so contributors share the convention.
---

# Dev Tasks Skill

Maintain structured task documentation that survives across Claude Code
sessions. Every non-trivial task gets a context folder. This is how you
prevent re-explaining the same constraints every session.

Dev-tasks state is repo-local but gitignored — working notes tied to
this checkout, not project artifacts, and never committed. Route other
kinds of persistence elsewhere: personal lessons and preferences that
span repos go to Claude Code's native auto-memory (user-level);
team-enforceable rules go through the sharpen skill into committed
artifacts (CLAUDE.md / `.claude/rules/`). Dev-tasks holds task-scoped
working state only: plan, context, and task list.

## When to Create Dev Tasks

- Starting any feature that will take more than one session
- When the user says "let's plan" or "I want to work on X"
- When resuming a task that already has dev task files
- When the user references a task by name

To create a task:

1. Choose a category (see [Choosing a Category](#choosing-a-category)).
2. Get today's UTC date with `date -u +%Y-%m-%d`.
3. Build the path `.claude/dev/<category>/<YYYY-MM-DD>-<slug>/` where `<slug>`
   is a kebab-case description of the work.
4. Create `plan.md`, `context.md`, and `tasks.md` in that folder.

## Directory Structure

```
.claude/dev/<category>/<YYYY-MM-DD>-<slug>/
├── plan.md       # The accepted implementation plan
├── context.md    # Decisions, constraints, gotchas — append-only audit trail
└── tasks.md      # Checklist of work items with status & log of actions performed under tasks as subbullets
```

Categories (default set): `features/`, `refactors/`, `fixes/`, `chores/`, `research/`

- `<YYYY-MM-DD>` is the **folder creation date, set once and never changed**.
  Do not rename folders as time passes — the date records when the task began.
  Get it with `date -u +%Y-%m-%d`.
- `<slug>` is a kebab-case description of the work (e.g. `datetime-package`).
- Worked example: a request to refactor the datetime package on 2026-06-28 →
  `.claude/dev/refactors/2026-06-28-datetime-package/`.

Ensure `.claude/dev/` is listed in the project's `.gitignore` (these are working
notes, not project artifacts) — add it if it isn't already.

Optional auxiliary files, used as needed: `HANDOFF.md` (knowledge transfer to the
next session) and `TARGET_STATE.md` (schema/architecture design doc).

### Choosing a Category

Pick the category that matches the nature of the work. The set aligns with the
conventional-commit types used by the sibling `commit-message` skill, so the
choice is predictable:

| Category     | Use for                                      | ~commit type |
|--------------|----------------------------------------------|--------------|
| `features/`  | New capability or feature                    | `feat`       |
| `fixes/`     | Bug fixes                                     | `fix`        |
| `refactors/` | Behavior-preserving code changes              | `refactor`   |
| `chores/`    | Tooling, dependencies, build, maintenance     | `chore`      |
| `research/`  | Investigations, spikes, design exploration    | (none)       |

- Default to these five. **Spikes always go in `research/`.**
- Only create a new top-level category when a task genuinely fits none of these.

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

**`context.md` is append-only.** It is the audit trail of what was accomplished
across sessions. Never delete or overwrite existing content — only add to it.
When something changes (a constraint shifts, a gotcha is resolved, code moves),
record it as a *new* entry (e.g. a dated Session Log line or a "Correction:"
note), leaving the original in place. Timestamp entries in **UTC ISO 8601**
(e.g. `2026-06-28T14:30:00Z`); get the current value with
`date -u +%Y-%m-%dT%H:%M:%SZ` rather than guessing.

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
- [YYYY-MM-DDThh:mm:ssZ]: [what was accomplished, what's left]
```

### tasks.md

```markdown
# [Task Name] — Tasks

## In Progress
- [ ] [task description]
    - [x] Step 1. [Exact changes made]
    - [ ] Step 2. [Exact changes made]

## Done
- [x] [YYYY-MM-DDThh:mm:ssZ] [task description] — [any notes]
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

1. Locate the task folder. Because folders are nested under a category and
   prefixed with a date, search across categories rather than guessing a path:
   glob `.claude/dev/*/*<slug>*/` (and the legacy flat `.claude/dev/<name>/` for
   back-compat). If multiple match, prefer the most recent date or ask the user.
2. Read all three files in the task folder
3. Summarize current status: what's done, what's in progress, what's next
4. Check if the plan is still accurate given any code changes since last session
5. If plan has drifted from code, flag the discrepancies
6. Continue from where the last session left off

Legacy flat-layout folders (`.claude/dev/<name>/`) remain valid and are read in
place — do not forcibly migrate them.

## Session End Behavior

Before ending a session or switching tasks:

1. Update `tasks.md` with completed and remaining items
2. Append any gotchas discovered to `context.md` (never overwrite)
3. Append a session log entry to `context.md` with a UTC ISO 8601 timestamp and a summary
4. If the plan changed during the session, update `plan.md`

## Rules

- Dev task files are living documents — update them as you learn, not just at start
- **`context.md` is append-only — never delete or overwrite its content.** It is the
  session audit trail; record corrections and new findings as new entries, preserving
  the original.
- Keep `plan.md` concise — if it exceeds 200 lines, the task should be split
- `context.md` is the place for things you'd forget between sessions
- `tasks.md` should be copy-pasteable as a PR checklist
- Never delete session log entries — they're a breadcrumb trail
- Timestamp session notes (`context.md`) and completed tasks (`tasks.md`) in **UTC
  ISO 8601** (e.g. `2026-06-28T14:30:00Z`); obtain it with `date -u +%Y-%m-%dT%H:%M:%SZ`,
  and convert any relative dates to absolute. Folder date prefixes use `date -u +%Y-%m-%d`.
- If the user says "catch me up" or "what were we working on", read the dev
  task files and summarize
- `.claude/dev/` is gitignored (working notes, not project artifacts). The relationship
  is one-way, like Jira: dev-task files may reference the codebase, but committed code,
  comments, and docs must never reference `.claude/dev/` paths
