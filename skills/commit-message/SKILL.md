---
name: commit-message
description: >
  Generate git conventional commit messages from staged changes or diffs. Produces
  structured commit messages with type, optional scope, subject, and body.
  Use when committing code or when asked to write a commit message.
---

# Commit Message Skill

Generate commit messages following the Conventional Commits specification.

## Format

For most commit messages:
```
<type>(<scope>): <subject>

<body>
```

For breaking changes & deprecation:
```
<type>(<scope>)!: <subject>

<body>
```

## Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `style` | Formatting, whitespace, missing semicolons (no logic change) |
| `docs` | Documentation only changes |
| `test` | Adding or updating tests |
| `chore` | Build process, dependency updates, tooling |
| `perf` | Performance improvement |
| `ci` | CI/CD configuration changes |

## Scope

Derive the scope from the area of the codebase changed. Examples:

- `feat(auth): add refresh token rotation`
- `refactor(api): extract error handling middleware`
- `docs(api): update authentication endpoint descriptions`
- `chore(deps): bump express to 5.1.0`

Use `!` for breaking changes and deprecation:

- `feat(api)!: migrate from firebase database integration to supabase`

If changes span multiple areas, use the primary area or omit scope.

If the project uses an external tracker (Jira, GitHub issues), its ticket
IDs may be included:
- `feat(AE-11458): add new oauth provider`
- `fix(api): handle empty cart checkout (#142)`

## Dev-Tasks Separation

`.claude/dev/` task files (the dev-tasks skill) are internal tracking
state, like a local Jira — not code. Commit messages carry **zero trace**
of the dev-tasks system:

- No `.claude/dev/` paths
- No dev-tasks task IDs (e.g., roadmap numbers like `2.1.2`)
- No bookkeeping mentions ("update dev-tasks", "sync task context")

When a dev-tasks context-file update rides along with its code commit,
stage it silently — the message describes the code change only. External
tracker IDs (real Jira keys, GitHub issues) remain fine.

## Subject Line Rules

- Imperative mood: "add" not "added" or "adds"
- No period at the end
- Max 72 characters
- Lowercase first letter after the colon
- Should complete the sentence: "If applied, this commit will ___"

## Body Rules

- Blank line between subject and body
- Wrap at 72 characters
- Explain WHAT changed and WHY, not HOW (the diff shows how)
- Reference related external issues/tickets if applicable (never dev-tasks)
- For breaking changes, add `BREAKING CHANGE:` footer

## Process

1. Read the staged diff (`git diff --staged`) or the diff provided by the user
2. Identify the primary type of change
3. Determine the scope from the files/modules changed
4. Write a clear subject line
5. If the change is non-trivial, add a body explaining the motivation

## Examples

**Simple feature:**
```
feat(api): add new oauth provider

Adds Google OAuth 2.0 provider to allow users to sign-in with their Google accounts.
```

**Bug fix:**
```
fix(auth): clear tokens on 401 before redirect

The onUnauthorized callback was redirecting to sign-in before
clearing the auth store, causing a brief flash of the previous
authenticated state on the sign-in screen.
```

**Dependency update:**
```
chore(deps): upgrade react-native-reanimated to 3.17.1

Fixes the shared value crash on Android API 34 reported in #142.
```

**Multi-file refactor:**
```
refactor: extract validation utils from form components

Moves inline validation logic from checkout-form.tsx
and register-form.tsx into utils/validation.ts. No behavior change —
all validation rules are identical to before.
```

## Anti-Patterns (never do these)

- `fix: stuff` — too vague
- `Updated files` — not conventional commits format
- `feat: add new feature for the dashboard screen that lets users...` — too long
- `fix(bug): fix bug` — scope and subject are redundant
- `WIP` or `temp` — never commit these
- `docs(dev-tasks): update task context` — dev-tasks state never appears
  in commit messages
