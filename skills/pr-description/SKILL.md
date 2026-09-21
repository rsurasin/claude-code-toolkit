---
name: pr-description
description: >
  Generate structured pull request descriptions from diffs or branch changes.
  Produces a clear summary, risk assessment (Low/Medium/High/Critical) based
  on verification evidence and decision reversibility, change list, testing
  notes, and review guidance. Use when creating a PR or when asked to write
  a PR description.
---

# PR Description Skill

Generate clear, structured pull request descriptions that help reviewers
understand what changed, why, how risky it is, and how to verify it.

## Process

1. Read the diff: `git diff main...HEAD` (or the target branch)
2. Read any related CLAUDE.md rules for context
3. Identify the category and scope of changes
4. Gather verification evidence from the session (see Risk Assessment Rubric)
5. Generate the description

## Template

```markdown
## Summary

[1-3 sentences: what this PR does and why. Should answer "what problem does
this solve?" without requiring the reviewer to read the diff.]

## Risk Assessment

**Risk Level: [LOW / MEDIUM / HIGH (unverified) / HIGH (findings) / CRITICAL]**

**Reversibility:** [Two-way door — revertible with a simple `git revert`,
no data migrations, no published API/schema/infra changes | One-way door —
state why: migration, deletion, published contract, irreversible infra]

| Dimension | Verified By | Result |
|-----------|-------------|--------|
| Tests | [repo's qa/test skill or agent, or test command] | [PASS / ISSUES (severity) / NOT RUN] |
| Runtime verification | [repo's verify skill or agent] | [PASS / ISSUES (severity) / NOT RUN] |
| Code review | `code-reviewer` agent | [PASS / ISSUES (severity) / NOT RUN] |
| Security | `security-auditor` agent | [PASS / ISSUES (severity) / NOT RUN] |
| Performance | `perf-audit` agent | [PASS / ISSUES (severity) / NOT RUN] |
| Docs | `doc-sync` agent | [PASS / ISSUES (severity) / NOT RUN] |

[1-2 sentences justifying the level: which floors applied, what gaps or
findings exist, and what would lower the risk.]

## Changes

[Grouped by area, not by file. Each item explains the *what* and *why*,
not just the file name.]

### [Area 1, e.g., "Authentication"]
- [Change description with motivation]
- [Change description with motivation]

### [Area 2, e.g., "API Layer"]
- [Change description with motivation]

## Testing

[How to verify this works. Be specific enough that someone unfamiliar with
the feature can test it.]

1. [Step 1]
2. [Step 2]
3. [Expected result]

## Screenshots / Recordings

[If UI changed, note that screenshots should be added. If no UI change,
say "N/A — no UI changes".]

## Notes for Reviewers

[Anything the reviewer should pay special attention to: tricky logic,
intentional deviations from patterns, known limitations, follow-up work.]

## Checklist

- [ ] Tests added/updated for changed logic
- [ ] Documentation updated (if public API changed)
- [ ] No secrets or credentials in the diff
- [ ] Breaking changes noted (if any)
```

Extend this checklist with project-specific items in your `.github/PULL_REQUEST_TEMPLATE.md` or `.claude/rules/`.

## Risk Assessment Rubric

The risk level is computed from two inputs: **reversibility** of the change
and **verification evidence** from this session. Fill the evidence table from
what actually ran — never from what should have run.

### Levels

- **LOW** — Two-way door AND every dimension ran clean: tests pass
  end-to-end, and code review, security, performance, and docs checks all
  ran with no issues found.
- **MEDIUM** — Two-way door with minor gaps: a non-critical dimension
  (docs, performance, or runtime verification) was `NOT RUN`, or only
  LOW-severity findings exist and all are acknowledged in the description.
- **HIGH** — One-way door with clean checks, OR significant verification
  gaps (tests or security NOT RUN), OR unresolved MEDIUM/HIGH-severity
  findings. Qualify the label so reviewers can tell the cause apart:
  **HIGH (unverified)** when driven by `NOT RUN` gaps or a one-way door
  with clean checks; **HIGH (findings)** when driven by unresolved
  findings.
- **CRITICAL** — One-way door combined with gaps or findings, OR a critical
  security vulnerability, OR a serious performance regression, OR
  verification was largely skipped.

### Floors (a floor can only raise the level, never lower it)

- Tests or Security `NOT RUN` ⇒ at best HIGH
- One-way door ⇒ at best HIGH
- Critical security finding or serious performance regression ⇒ CRITICAL
- Failing tests ⇒ CRITICAL

### Verdict normalization

Verification sources report in different vocabularies. Map them as:

| Source | Reported | Normalized |
|--------|----------|------------|
| Test run | all tests pass | PASS |
| Test run | any failure or blocked run | ISSUES (HIGH) |
| Repo verify skill/agent | pass | PASS |
| Repo verify skill/agent | pass with warnings | ISSUES (LOW) |
| Repo verify skill/agent | fail | ISSUES (HIGH) |
| code-reviewer / security-auditor / perf-audit / doc-sync | no findings | PASS |
| code-reviewer / security-auditor / perf-audit / doc-sync | findings | ISSUES (highest severity found) |

### Rules of evidence

- Tests and runtime verification come from the **target repo's own** qa/verify
  skill or agent (defined in that repo's `.claude/`), or from the test
  commands documented in its CLAUDE.md. The toolkit deliberately does not
  ship generic test/verify agents.
- When this skill is invoked from `commit-push-pr` ship mode, the evidence
  table comes from that single whole-branch gate run (`<base>...HEAD`).
  Findings above the fix threshold were already fixed and their gates
  re-run, so surviving ISSUES entries should be LOW and acknowledged.
- When this skill is invoked standalone (outside `/commit-push-pr`), use
  whatever evidence already exists in the session. Mark everything else
  `NOT RUN` and let the floors raise the level. Never claim LOW without
  evidence for every dimension.
- Reversibility is judged from the diff: data migrations, schema changes,
  deleted data, published API/package changes, and infra mutations are
  one-way doors. Pure code changes behind an unreleased surface are two-way.

## Rules

- The summary should be understandable by a PM, not just engineers
- The risk assessment is evidence-based — report what ran, not what should have run
- Group changes by logical area, not by file
- Testing steps should be copy-pasteable
- If the PR is large (>15 files), add a "How to Review" section suggesting
  an order to read the files
- If the PR introduces a new pattern, call it out explicitly
- Reference external tracker IDs (Jira keys, GitHub issues) if applicable —
  never dev-tasks task IDs or `.claude/dev/` paths; dev-tasks is internal
  tracking state and PR descriptions carry no trace of it
- Keep it concise — a PR description longer than the diff is a red flag
