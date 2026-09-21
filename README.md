# claude-code-toolkit

Universal Claude Code plugin with agents, skills, commands, and hooks for
development workflow automation. Works across any language, framework, or
project type — Go, Python, Rust, TypeScript, Java, Ruby, C#, PHP, and more.

---

## What's Inside

### Agents (7) — auto-dispatched by Claude based on your request

| Agent | What It Does |
|-------|--------------|
| `code-reviewer` | Parallel 4-axis review: bugs, security, performance, conventions |
| `security-auditor` | Deep OWASP-aligned security audit with language-aware vulnerability detection |
| `debugger` | Systematic hypothesis-driven bug investigation |
| `perf-audit` | Runtime perf: startup time, re-renders, N+1 queries, bundle size, memory, concurrency |
| `doc-sync` | Audits docs against code, fixes documentation drift |
| `claude-sync` | Audits `.claude/` config files against codebase |
| `cross-repo-audit` | Compares two related repos for contract breaks and doc conflicts |

> Test running and app verification are intentionally **not** shipped as
> generic agents — define a qa/verify skill in each repo's `.claude/`
> directory instead. See [Migrating from v2](#migrating-from-v2).

### Skills (6) — auto-loaded by Claude when context matches

| Skill | What It Does |
|-------|--------------|
| `commit-message` | Generates conventional commit messages from diffs |
| `pr-description` | Generates structured PR descriptions with a risk assessment (Low/Medium/High/Critical) computed from verification evidence and reversibility |
| `commit-push-pr` | Two-mode inner loop: fast atomic task commits (no gates), then gated PR creation — quality gates (repo tests/verify + review, security, perf, docs agents) run once over the full branch diff, findings are fixed, and the PR opens with a risk assessment |
| `dev-tasks` | Maintains category-organized task files (plan/context/tasks) under `.claude/dev/` for cross-session memory |
| `sharpen` | Proactively encodes a mistake or inefficiency (agent- or user-detected) into a permanent harness improvement — test, lint rule, CLAUDE.md entry — always with user consent before touching the harness |
| `review-plan` | Fresh-context staff engineer critique of a plan before execution, with a LOW/MEDIUM/HIGH/CRITICAL risk level on the pr-description scale — fires proactively before non-trivial plans |

### Bundled (already in Claude Code — use alongside this plugin)

| Command | When to Use |
|---------|-------------|
| `/simplify` | After implementing a feature — reviews for code reuse, quality, efficiency |
| `/batch` | Large-scale refactors across many files — parallelizes into 5-30 units |
| `/btw` | While Claude is working, provide it additional input |
| `/code-review` | Review the working diff; `/code-review ultra` runs a multi-agent cloud review of the branch or a PR (user-triggered, billed) |
| `security-review` | Diff-scoped security review of the current branch's pending changes — a user-facing complement to `security-auditor`, not a gate replacement |
| `/run` | Launch and drive the project's app to confirm a change works — build repo-local verify skills on top of it |
| `/init` | Generate a CLAUDE.md for a repo |
| Auto-memory / `/remember` | Personal, user-level lessons and preferences (not committed, invisible to teammates) — vs `sharpen` (repo-shared rules) and `dev-tasks` (repo-shared task state) |

> Availability varies by Claude Code version and surface. Where a native
> skill overlaps a toolkit agent (`/code-review` ↔ `code-reviewer`,
> `security-review` ↔ `security-auditor`), they complement rather than
> replace each other: the ship-mode quality gates deliberately use the
> dedicated toolkit agents, because a skill runs in the context of the
> agent that just wrote the code (self-review bias), while a spawned
> agent reviews the diff cold and returns a normalized verdict.

---

## Language & Framework Support

Every agent auto-detects your project type and adapts accordingly. No configuration needed.

| Category | Supported |
|----------|-----------|
| **Languages** | Go, TypeScript/JavaScript, Python, Rust, Java, Kotlin, Ruby, C#, PHP, Elixir, Swift, C/C++ |
| **Frontend** | React, Next.js, Vue, Nuxt, Svelte, Angular, React Native, Expo |
| **Backend** | Express, Fastify, Django, Flask, FastAPI, Rails, Spring Boot, Laravel, Phoenix, ASP.NET, Gin, Actix |
| **Testing** | go test, jest, vitest, pytest, cargo test, JUnit, Gradle, RSpec, PHPUnit, dotnet test, mix test, swift test |
| **Databases** | PostgreSQL, MySQL, SQLite, MongoDB — via any ORM or query builder |
| **Build tools** | npm, Cargo, Gradle, Maven, Make, CMake, Mix, pip, Bundler, Composer |

Monorepos with multiple sub-projects are handled by running parallel subagents per project.

---

## Installation

### Option 1: Plugin Install (recommended)

In Claude Code:

```
/plugin install claude-code-toolkit@rahul-claude-code-toolkit
```

Choose **"Install for you (user scope)"** for global availability across all projects.

> **First time?** Register the marketplace first:
> ```
> /plugin marketplace add https://github.com/rsurasin/claude-code-toolkit
> ```

### Option 2: Local (for development/testing)

```bash
claude --plugin-dir /path/to/claude-code-toolkit
```

### NixOS Notes

Claude Code stores plugins in `~/.claude/plugins/cache/` (mutable home
directory state). This works fine on NixOS — just ensure `~/.claude/` isn't
managed as immutable by home-manager. Run `/plugin install` once after
`claude` is available in your shell.

---

## MCP Server Configuration (optional, for mobile verification)

If your repo-local verification skill covers a **mobile app** (React
Native / Expo), pairing it with MCP servers enables native device
automation:

### Android (works on Linux/NixOS)

```json
{
  "mcpServers": {
    "mobile-mcp": {
      "command": "npx",
      "args": ["-y", "@mobilenext/mobile-mcp@latest"]
    }
  }
}
```

Requires: ADB installed and in PATH, Android emulator running or device connected.

### iOS (macOS only)

```json
{
  "mcpServers": {
    "expo-mcp": {
      "type": "url",
      "url": "https://mcp.expo.dev/sse"
    },
    "xc-mcp": {
      "command": "npx",
      "args": ["xc-mcp"]
    }
  }
}
```

Requires: Xcode installed, iOS simulator.
Also install: `npx expo install expo-mcp --dev` in your project.
Start dev server with: `EXPO_UNSTABLE_MCP_SERVER=1 npx expo start`

### Expo MCP (cross-platform)

```json
{
  "mcpServers": {
    "expo": {
      "type": "url",
      "url": "https://mcp.expo.dev/sse"
    }
  }
}
```

Provides documentation search, dependency management, and (with local
capabilities) testID-based element interaction.

---

## Recommended Hooks (per-repo, not included in plugin)

Hooks should live in each project's `.claude/hooks/hooks.json` — not in a
universal plugin. They should be precise, project-specific, and fail loud.

**PostToolUse** — format each file immediately after Claude writes it.
Use your project's actual formatter (prettier, gofmt, black, rustfmt, etc.).

**Stop** — run a fast compile/lint check when Claude finishes a task.
Scope to changed files only to keep it under a few seconds.

See the [Claude Code hooks documentation](https://code.claude.com/docs/en/hooks)
for configuration details.

---

## Usage

### Natural language — Claude routes automatically

```
"review my changes"              → code-reviewer agent
"why is this broken"             → debugger agent
"security audit"                 → security-auditor agent
"check performance"              → perf-audit agent
"sync the docs"                  → doc-sync agent
"update the claude config"       → claude-sync agent
"compare these two repos"        → cross-repo-audit agent
```

For "run the tests" or "verify the app works", define qa/verify skills in
each repo's `.claude/` directory — see [Migrating from v2](#migrating-from-v2).

### Skills activate automatically

```
"write a commit message"         → commit-message skill
"generate a PR description"      → pr-description skill (with risk assessment)
"commit this"                    → commit-push-pr skill (commit mode: fast atomic commit + push)
"ship this / open a PR"          → commit-push-pr skill (ship mode: gates over full branch diff, then PR)
"let's plan the logging feature" → dev-tasks skill (creates .claude/dev/features/<date>-<slug>/)
"catch me up"                    → dev-tasks skill (reads existing task files)
"no, do it this way" (a correction) → sharpen skill (proposes a harness rule; asks consent first)
```

### Slash commands

```
/sharpen                         → encode a mistake into a harness improvement (skill; also fires proactively)
/commit-push-pr                  → commit mode by default; ship mode (gates + PR) when asked to ship
/review-plan                     → critical plan review with risk level (skill; also fires proactively)
```

---

## Recommended Workflow

### Daily Development
1. Start session: `dev-tasks` (if picking up where you left off)
2. Plan: Use Plan Mode; the `review-plan` skill critiques the plan in a
   fresh-context subagent before executing (LOW/MEDIUM/HIGH/CRITICAL —
   CRITICAL means reconsider the approach)
3. Implement: Let Claude work in auto-accept mode
4. Polish: `/simplify` after completing a feature
5. Commit per task: `/commit-push-pr` — commit mode makes a fast, atomic
   commit (one task = one commit) and pushes; no gates, no PR
6. Ship: when the feature is complete, "open a PR" — ship mode runs the
   quality gates once over the full branch diff (your repo's qa/verify
   skills plus code review, security, performance, and docs agents), fixes
   any findings, then opens a PR with a risk assessment
7. Learn: the `sharpen` skill fires on any mistake or correction — agent-
   or user-detected — and proposes a permanent harness improvement,
   applying it only with your consent (`/sharpen` still works explicitly)

### Weekly Maintenance
1. `doc-sync` agent: "sync documentation with current code"
2. `claude-sync` agent: "update the .claude config files"
3. `cross-repo-audit` agent: "check consistency between repos"

### Before Releases
1. Your repo's qa skill/agent: "run all tests"
2. `security-auditor` agent: "full security audit"
3. `perf-audit` agent: "performance audit before release"
4. Your repo's verify skill/agent: "full verification"

---

## Directory Structure

```
claude-code-toolkit/
├── .claude-plugin/
│   ├── plugin.json                  # Plugin manifest
│   └── marketplace.json             # Distribution catalog
├── agents/
│   ├── code-reviewer.md             # Parallel multi-axis code review
│   ├── security-auditor.md          # Deep security audit (OWASP-aligned)
│   ├── debugger.md                  # Systematic bug investigation
│   ├── perf-audit.md                # Runtime performance analysis
│   ├── doc-sync.md                  # Documentation parity auditor
│   ├── claude-sync.md               # .claude directory updater
│   └── cross-repo-audit.md          # Cross-repo consistency checker
├── skills/
│   ├── commit-message/
│   │   └── SKILL.md                 # Git conventional commit generator
│   ├── pr-description/
│   │   └── SKILL.md                 # PR description generator w/ risk assessment
│   ├── commit-push-pr/
│   │   └── SKILL.md                 # Two-mode commit → PR inner loop
│   ├── dev-tasks/
│   │   └── SKILL.md                 # Task-specific cross-session context
│   ├── sharpen/
│   │   └── SKILL.md                 # Mistake → harness improvement loop (consent-gated)
│   └── review-plan/
│       └── SKILL.md                 # Fresh-context plan critique with risk rubric
├── evals/                           # Behavioral eval cases (claude plugin eval layout)
├── .gitignore
├── CLAUDE.md                        # Guidance for Claude Code working on this repo
└── README.md
```

The `dev-tasks` skill maintains its own working notes at runtime under a
gitignored `.claude/dev/` tree, organized by category and date:

```
.claude/dev/<category>/<YYYY-MM-DD>-<slug>/   # e.g. refactors/2026-06-28-datetime-package/
├── plan.md      # accepted implementation plan
├── context.md   # append-only audit trail (decisions, constraints, gotchas)
└── tasks.md     # work checklist with status & action log
```

Categories: `features/`, `refactors/`, `fixes/`, `chores/`, `research/`.

---

## Design Philosophy

This plugin is built on three principles observed across staff-level engineers:

1. **Feedback loops over configuration.** `/sharpen` encodes every mistake
   into a permanent improvement. Each session makes the next one more capable.

2. **Verification over trust.** The `/commit-push-pr` gates confirm the
   project actually works — via each repo's own qa/verify skills, which
   know the project's real commands — before a PR opens, and the PR carries
   a risk assessment showing exactly what was and wasn't verified.
   Verification 2-3x the quality of unverified work.

3. **Vanilla over complex.** The agents handle post-implementation analysis
   (review, test, debug, audit). They don't try to control how Claude
   implements features — that's what CLAUDE.md and Plan Mode are for.

---

## Migrating from v2

v3.0.0 removed the generic `qa-runner` and `verify-app` agents. They
auto-detected ecosystems to guess test/verify commands; a small skill in
each repo that knows the *actual* commands is more reliable and feeds the
`/commit-push-pr` quality gates directly.

Create per-repo skills like `.claude/skills/qa/SKILL.md`:

```markdown
---
name: qa
description: Run this project's test suite and report failures.
---

Run `make test` (unit + integration). On failure, report the failing test
names and the relevant output. Never edit tests to make them pass.
```

And `.claude/skills/verify/SKILL.md` with the project's real build/run/smoke
steps. `/commit-push-pr` discovers these automatically; if a repo has
neither, its gates are recorded as `NOT RUN`, which raises the PR's risk
level.

Also in v3.0.0: all three commands — `commit-push-pr`, `sharpen`, and
`review-plan` — moved from `commands/` to `skills/` (the `commands/`
directory is gone; the `/commit-push-pr`, `/sharpen`, and `/review-plan`
invocations still work). Note that the quality gates run at **PR
creation** (ship mode), not on every commit — intermediate task commits
stay fast and gate-free. `sharpen` now also fires proactively on detected
mistakes or inefficiencies, proposing harness improvements that are
applied only with explicit user consent. And `review-plan` rates plans
LOW/MEDIUM/HIGH/CRITICAL on the same scale as the pr-description risk
assessment, running its critique in a fresh-context subagent.

---

## Updating

Releasing a new version:

1. Bump `version` in `.claude-plugin/plugin.json` **and**
   `.claude-plugin/marketplace.json`
2. Commit and push to `main` — the marketplace serves the plugin from the
   GitHub repo, so unpushed local changes are never picked up

Then update the installed copy from the CLI:

```bash
claude plugin update claude-code-toolkit@rahul-claude-code-toolkit
```

(Restart Claude Code to apply. If the new version isn't found, refresh the
marketplace clone first: `claude plugin marketplace update
rahul-claude-code-toolkit`.)

Or from inside Claude Code:

```
/plugin → "Manage and uninstall plugins" → claude-code-toolkit → "Update now"
```

If installed via marketplace with auto-update enabled, Claude Code pulls
new versions on startup.
