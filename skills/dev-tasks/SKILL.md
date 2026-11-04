---
name: dev-tasks
description: >
  Maintain task-specific context files (plan, context, tasks) under
  .claude/dev/ that persist across Claude Code sessions, including each
  task's status from in progress through PR, merge, and release, and the
  steps a release must carry out. Use when starting a non-trivial feature,
  when resuming work from a previous session, when a release closes out
  merged work, or when asked to create or update development task
  documentation. Bridges the "daily amnesia" problem. Repo-local so
  contributors share the convention.
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
└── tasks.md      # Checklist of work items with status & log of actions performed under tasks as subbullets, then release steps and the status line
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

## Release Steps
- [ ] [what the release itself must do, e.g. "deploy together with org/frontend#56", "apply migration 0042_cap_sets", "turn on the streaks flag"; or "None"]

Status: in progress
```

The `## Release Steps` heading and the final `Status:` line are fixed: the
release closeout finds them by that exact heading and prefix, so never
rename them or move the status line off the last line. See
[Status and Release Tracking](#status-and-release-tracking).

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

At every session start, even when resuming a different task, also surface
pending release work:

- List every folder whose status is still `merged to <base> (<sha>), not
  released`, with its base branch and open release steps, so work waiting
  on a release stays visible:
  `grep -l '^Status: merged to' .claude/dev/*/*/tasks.md`
- Reconcile folders whose status is `PR open (#N)` with GitHub
  (`gh pr view <N> --json state,mergedAt,mergeCommit,baseRefName`) and
  apply the matching transition from
  [Status and Release Tracking](#status-and-release-tracking) if the PR
  has since merged or closed. If `gh` is unavailable, say the statuses
  could not be checked rather than guessing.

## Status and Release Tracking

The last line of `tasks.md` records where the work stands. It is exactly
one of these four states, with no other wording:

| Status line | Meaning | Set by |
|-------------|---------|--------|
| `Status: in progress` | Work under way, no open PR | Folder creation; `commit-push-pr` commit mode |
| `Status: PR open (#N)` | PR #N is open for this work | `commit-push-pr` ship mode, once the PR opens |
| `Status: merged to <base> (<sha>), not released` | PR merged into a non-default integration branch (e.g. `dev`) as short SHA `<sha>`; waiting on a release | Session-start reconcile, or the user reporting the merge |
| `Status: released (<release PR #N>, <date>)` | Reached the default branch in release PR #N on `<date>` (`date -u +%Y-%m-%d`), release steps done or handed off | `commit-push-pr` release mode, after the closeout |

Transitions:

- **PR merged into an integration branch**: `merged to <base> (<sha>),
  not released`.
- **PR merged straight into the default branch** (repos with no
  integration branch): the PR is its own release. Run the post-merge part
  of the [release closeout](#release-closeout) for it, then set
  `released (#N, <date>)`.
- **PR closed without merging**: back to `in progress`.
- Append every transition to `context.md`'s Session Log (e.g.
  `status: PR open (#41)`), so the PR number survives later transitions.

Record a release step under `## Release Steps` as soon as it is known
during implementation: a deploy that must pair with another repo's
change, a migration to apply, a flag to flip, a config value or secret to
set. Write each step so someone else could do it: what, where, and what
it must ship together with. Write `None` when the release needs nothing
beyond the merge.

### Release closeout

`commit-push-pr` release mode calls this when a PR from the integration
branch to the default branch opens or merges. This section is the folder
side of the release; the PR side lives in that skill.

1. **Map each PR in the release to its folder**, in this order: the
   `gh-<N>-` in the PR's branch name (`<type>/gh-<N>-<slug>`) matches the
   folder named `<YYYY-MM-DD>-gh-<N>-<slug>`; else an issue the PR closes
   matches a folder's `gh-<N>-`; else a folder whose Session Log records
   that PR number. Example: PR #41 from `feat/gh-31-password-reset` maps
   to `.claude/dev/features/2026-10-04-gh-31-password-reset/`. A PR with
   no folder is listed as "no task folder, release steps unknown" and the
   user is asked about it. A folder still `merged to <integration> ...,
   not released` that no PR in the release maps to is flagged too: it may
   have been reverted or missed.
2. **Gather** each mapped folder's `## Release Steps` and the issues its
   PR closes, for the release PR description.
3. **After the release PR merges**, for each folder: mark each release
   step done (`[x]`) or handed off (`- handed off to <who>, <date>`), set
   the status line to `released (#<release PR>, <date>)`, and append a
   Session Log entry to `context.md`.

Never mark a folder `released` before its release PR has merged.

## Session End Behavior

Before ending a session or switching tasks:

1. Update `tasks.md` with completed and remaining items, newly known
   release steps, and the status line
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
- `tasks.md` always ends with exactly one `Status:` line from the fixed set in
  [Status and Release Tracking](#status-and-release-tracking). A folder without one
  (created before this convention) counts as `in progress` and gets the line on its
  next update
- Never delete session log entries — they're a breadcrumb trail
- Timestamp session notes (`context.md`) and completed tasks (`tasks.md`) in **UTC
  ISO 8601** (e.g. `2026-06-28T14:30:00Z`); obtain it with `date -u +%Y-%m-%dT%H:%M:%SZ`,
  and convert any relative dates to absolute. Folder date prefixes use `date -u +%Y-%m-%d`.
- If the user says "catch me up" or "what were we working on", read the dev
  task files and summarize
- `.claude/dev/` is gitignored (working notes, not project artifacts). The relationship
  is one-way, like Jira: dev-task files may reference the codebase, but committed code,
  comments, and docs must never reference `.claude/dev/` paths. A release PR built from
  the closeout carries features, PR and issue numbers, and release steps, never folder
  paths, folder names, or dev-tasks task IDs
