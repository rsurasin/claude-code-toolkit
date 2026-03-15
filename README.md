# claude-code-toolkit

Universal Claude Code plugin with agents, skills, commands, and hooks for
development workflow automation. Works across any language, framework, or
project type — Go, Python, Rust, TypeScript, Java, Ruby, C#, PHP, and more.

---

## What's Inside

### Agents (9) — auto-dispatched by Claude based on your request

| Agent | What It Does |
|-------|--------------|
| `code-reviewer` | Parallel 4-axis review: bugs, security, performance, conventions |
| `qa-runner` | Runs test suite across 12+ ecosystems, classifies failures, suggests fixes |
| `security-auditor` | Deep OWASP-aligned security audit with language-aware vulnerability detection |
| `debugger` | Systematic hypothesis-driven bug investigation |
| `perf-audit` | Runtime perf: re-renders, N+1 queries, bundle size, memory, concurrency |
| `verify-app` | Tiered verification: static checks → runtime validation → manual test checklist |
| `doc-sync` | Audits docs against code, fixes documentation drift |
| `claude-sync` | Audits `.claude/` config files against codebase |
| `cross-repo-audit` | Compares two related repos for contract breaks and doc conflicts |

### Skills (3) — auto-loaded by Claude when context matches

| Skill | What It Does |
|-------|--------------|
| `commit-message` | Generates conventional commit messages from diffs |
| `pr-description` | Generates structured PR descriptions with testing steps |
| `dev-tasks` | Maintains task-specific context files for cross-session memory |

### Commands (3) — invoked via `/command-name`

| Command | What It Does |
|---------|--------------|
| `/sharpen` | Encode a mistake into a permanent harness improvement (test, lint rule, CLAUDE.md entry) |
| `/commit-push-pr` | Full inner loop: stage, commit, push, open PR — one command |
| `/review-plan` | Critical staff engineer review of a plan before execution |

### Bundled (already in Claude Code — use alongside this plugin)

| Command | When to Use |
|---------|-------------|
| `/simplify` | After implementing a feature — reviews for code reuse, quality, efficiency |
| `/batch` | Large-scale refactors across many files — parallelizes into 5-30 units |
| `/btw` | While Claude is working, provide it additional input |

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

### Option 3: Local (for development/testing)

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

The `verify-app` agent works out of the box for most project types. For
**mobile app verification** (React Native / Expo), pairing with MCP servers
enables native device automation:

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
"run the tests"                  → qa-runner agent
"verify the app works"           → verify-app agent
"why is this broken"             → debugger agent
"security audit"                 → security-auditor agent
"check performance"              → perf-audit agent
"sync the docs"                  → doc-sync agent
"update the claude config"       → claude-sync agent
"compare these two repos"        → cross-repo-audit agent
```

### Skills activate automatically

```
"write a commit message"         → commit-message skill
"generate a PR description"      → pr-description skill
"let's plan the logging feature" → dev-tasks skill (creates task context files)
"catch me up"                    → dev-tasks skill (reads existing task files)
```

### Slash commands

```
/sharpen                         → encode a mistake into a harness improvement
/commit-push-pr                  → stage, commit, push, open PR
/review-plan                     → critical review before executing a plan
```

---

## Recommended Workflow

### Daily Development
1. Start session: `/catchup` (or `/clear` + `/catchup` for fresh context)
2. Plan: Use Plan Mode, then `/review-plan` before executing
3. Implement: Let Claude work in auto-accept mode
4. Polish: `/simplify` after completing a feature
5. Test: `qa-runner` agent will run all tests
6. Verify: `verify-app` agent for runtime checks
7. Ship: `/commit-push-pr` to stage, commit, push, and open PR
8. Learn: `/sharpen` after any mistake

### Weekly Maintenance
1. `doc-sync` agent: "sync documentation with current code"
2. `claude-sync` agent: "update the .claude config files"
3. `cross-repo-audit` agent: "check consistency between repos"

### Before Releases
1. `qa-runner` agent: "run all tests"
2. `security-auditor` agent: "full security audit"
3. `perf-audit` agent: "performance audit before release"
4. `verify-app` agent: "full verification"

---

## Directory Structure

```
claude-code-toolkit/
├── .claude-plugin/
│   ├── plugin.json                  # Plugin manifest
│   └── marketplace.json             # Distribution catalog
├── agents/
│   ├── code-reviewer.md             # Parallel multi-axis code review
│   ├── qa-runner.md                 # Test suite runner & failure analyzer
│   ├── security-auditor.md          # Deep security audit (OWASP-aligned)
│   ├── debugger.md                  # Systematic bug investigation
│   ├── perf-audit.md                # Runtime performance analysis
│   ├── verify-app.md                # Tiered project verification
│   ├── doc-sync.md                  # Documentation parity auditor
│   ├── claude-sync.md               # .claude directory updater
│   └── cross-repo-audit.md          # Cross-repo consistency checker
├── skills/
│   ├── commit-message/
│   │   └── SKILL.md                 # Git conventional commit generator
│   ├── pr-description/
│   │   └── SKILL.md                 # Structured PR description generator
│   └── dev-tasks/
│       └── SKILL.md                 # Task-specific cross-session context
├── commands/
│   ├── sharpen.md                   # Mistake → harness improvement loop
│   ├── commit-push-pr.md            # Full commit → PR inner loop
│   └── review-plan.md               # Critical plan review before execution
├── .gitignore
└── README.md
```

---

## Design Philosophy

This plugin is built on three principles observed across staff-level engineers:

1. **Feedback loops over configuration.** `/sharpen` encodes every mistake
   into a permanent improvement. Each session makes the next one more capable.

2. **Verification over trust.** `verify-app` confirms the project actually works,
   not just that the code looks right. Verification 2-3x the quality of
   unverified work.

3. **Vanilla over complex.** The agents handle post-implementation analysis
   (review, test, debug, audit). They don't try to control how Claude
   implements features — that's what CLAUDE.md and Plan Mode are for.

---

## Updating

Bump `version` in `.claude-plugin/plugin.json` when pushing changes.

If installed via marketplace with auto-update enabled, Claude Code pulls
new versions on startup. Otherwise:

```
/plugin → "Manage and uninstall plugins" → claude-code-toolkit → "Update now"
```
