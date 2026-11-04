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

**Risk Level: [LOW / MEDIUM / HIGH (unverified) / HIGH (findings) / HIGH (findings, unverified) / CRITICAL]**

**Reversibility:** [Two-way door — revertible with a simple `git revert`,
no data migrations, no published API/schema/infra changes | One-way door —
state why: migration, deletion, published contract, irreversible infra]

| Dimension | Verified By | Result |
|-----------|-------------|--------|
| Tests | [repo's qa/test skill or agent, or test command] | [PASS / ISSUES (severity) / NOT RUN] |
| Runtime verification | [repo's verify skill or agent] | [PASS / ISSUES (severity) / NOT RUN] |
| Code review | `code-reviewer` agent | [PASS / ISSUES (severity) / NOT RUN] |
| Code review (built-in) | `/code-review` skill | [PASS / ISSUES (severity) / NOT RUN / N/A] |
| Security | `security-auditor` agent | [PASS / ISSUES (severity) / NOT RUN] |
| Security (built-in) | `/security-review` skill | [PASS / ISSUES (severity) / NOT RUN / N/A] |
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

`NOT RUN` means the check exists but didn't run (no repo qa skill, a
gate that errored or was skipped). `N/A` means a built-in skill isn't
available in the running Claude Code version. Both are verification
gaps.

### Levels

- **LOW** — Two-way door AND every dimension ran clean: tests pass
  end-to-end, and every review, security, performance, and docs check
  (agents and built-in skills alike) ran with no issues found. No
  `NOT RUN` and no `N/A` rows.
- **MEDIUM** — Two-way door with minor gaps: a non-critical dimension
  (docs, performance, runtime verification, one of the two code-review
  rows while the other ran, or one of the two security rows while the
  other ran) was `NOT RUN` or `N/A`, or only LOW-severity findings exist
  and all are acknowledged in the description.
- **HIGH** — One-way door with clean checks, OR significant verification
  gaps (tests `NOT RUN`, or neither security gate ran, or neither
  code-review gate ran), OR unresolved MEDIUM/HIGH-severity findings.
  Qualify the label so reviewers can tell the cause apart:
  **HIGH (unverified)** when driven by `NOT RUN` / `N/A` gaps or a
  one-way door with clean checks; **HIGH (findings)** when driven by
  unresolved findings; **HIGH (findings, unverified)** when both apply.
- **CRITICAL** — One-way door combined with gaps or findings, OR an
  introduced CRITICAL finding of any kind (security, data loss, a core
  path broken for every user), OR a serious performance regression, OR
  verification was largely skipped.

### Floors (a floor can only raise the level, never lower it)

- Any dimension `NOT RUN` or `N/A` ⇒ at best MEDIUM (a gap is never LOW)
- Tests `NOT RUN` ⇒ at best HIGH
- Security and Security (built-in) both `NOT RUN` or `N/A` ⇒ at best HIGH
- Code review and Code review (built-in) both `NOT RUN` or `N/A` ⇒ at
  best HIGH
- One-way door ⇒ at best HIGH
- An introduced CRITICAL finding of any kind, or any CRITICAL security
  finding outside ship mode (untagged, or from a full-scope audit), or
  a serious performance regression ⇒ CRITICAL. Only in ship mode does
  an existing critical security finding skip this floor — ship mode
  makes it a hard stop instead
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
| `/code-review` | no findings | PASS |
| `/code-review` | findings | ISSUES (highest severity assigned during triage) |
| `/security-review` | no findings | PASS |
| `/security-review` | findings | ISSUES (highest severity after triage) |
| `/code-review` / `/security-review` | skill unavailable in this Claude Code version | N/A |
| any gate | errored, timed out, or skipped | NOT RUN |

`/code-review` reports findings without a severity, and `perf-audit` and
`doc-sync` use their own labels; ship mode's triage step maps all of
them onto the Severity Scale below before normalizing. When ship mode's
triage has tagged findings **introduced** or **existing**, the verdict's
severity comes from introduced findings only (LOW included) — a gate
with only existing findings is PASS. Existing non-security findings are
listed under Notes for Reviewers (with issue links when filed); existing
security findings, at any severity, are reported to the user, not
published in the PR body. A finding merged across reviewers appears in
every source's row at the severity that source gave it (triage-assigned
for `/code-review`); the merged, highest severity drives the fix loop
and the risk level.

### Severity Scale

All reviewers' findings map onto one scale, defined by consequence — not
by how small the fix is or how unlikely the trigger feels. The
`code-reviewer` agent uses the same definitions.

- **CRITICAL** — exploitable security vulnerability, data loss or
  corruption, or a change that breaks a core path for every user.
- **HIGH** — a reachable path produces a wrong result: wrong persisted
  data, a race that leaves inconsistent state, a security or
  authorization boundary crossed, a crash or unhandled error on a normal
  request, a removed invariant nothing re-establishes, or a caller the
  change breaks.
- **MEDIUM** — wrong behavior under specific but plausible conditions
  (an edge input, a time zone, a retry), a changed path with no test in
  a codebase that has tests, a hard project-rule violation, or
  docs/comments that now contradict the code. In a codebase with no
  tests, missing coverage is one LOW summary note, not a MEDIUM per path.
- **LOW** — hardening, simplification, reuse, style, naming, or a
  performance improvement without evidence of user-visible impact.

A one-line fix for a real race is still HIGH. Lower a severity for
likelihood only when the triggering state is shown to be unreachable.

### Rules of evidence

- Tests and runtime verification come from the **target repo's own** qa/verify
  skill or agent (defined in that repo's `.claude/`), or from the test
  commands documented in its CLAUDE.md. The toolkit deliberately does not
  ship generic test/verify agents.
- When this skill is invoked from `commit-push-pr` ship mode, the evidence
  table comes from that single whole-branch gate run (`<base>...HEAD`)
  after its triage step. Introduced findings above the fix threshold were
  already fixed and their gates re-run, so surviving ISSUES entries
  should be LOW, or at most one MEDIUM, and acknowledged.
- When this skill is invoked standalone (outside `/commit-push-pr`), use
  whatever evidence already exists in the session. Mark everything else
  `NOT RUN` (or `N/A` for a built-in skill this Claude Code version
  doesn't have) and let the floors raise the level. Never claim LOW without
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
- Never write the location, class, or exploit path of a security finding
  the diff didn't introduce — or of any security finding from a
  full-scope audit — into the description. Write "security finding(s)
  reported privately", tell the user the details directly, and
  recommend a private channel such as a GitHub security advisory. When
  a security finding's provenance is unclear (no introduced/existing
  tag), treat it as not introduced
- Keep it concise — a PR description longer than the diff is a red flag
