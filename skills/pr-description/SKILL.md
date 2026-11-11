---
name: pr-description
description: >
  Generate structured pull request descriptions from diffs or branch changes.
  Produces a summary stating the current behavior, risk assessment
  (Low/Medium/High/Critical) based on verification evidence and decision
  reversibility, a change list led by the new behavior, testing notes,
  screenshots or runtime output, and review guidance. Use when creating a
  PR or when asked to write a PR description.
---

# PR Description Skill

Generate clear, structured pull request descriptions that help reviewers
understand what changed, why, how risky it is, and how to verify it.

## Process

1. Read the diff: `git diff main...HEAD` (or the target branch)
2. Read any related CLAUDE.md rules for context
3. Identify the category and scope of changes
4. Gather verification evidence from the session (see Risk Assessment
   Rubric), including any runtime-verification output for the
   Screenshots / Output section
5. Collect the GitHub issues this PR resolves. Sources: issues the user named,
   and a `gh-<N>-` segment in the branch name (`git rev-parse --abbrev-ref HEAD`,
   e.g. `feat/gh-31-password-reset`) — use it only when `<N>` is digits only.
   GitHub links issues through the PR body, not the branch name, so the
   description must carry the reference:
   - Write `Closes #N` (on its own line at the end of the Summary) when the
     user named the issue, or the branch number agrees with the active
     dev-tasks folder's `gh-<N>-` — two sources confirm it.
   - If the branch number is the only source, or the PR only partly
     addresses the issue, write `Refs #N` instead and mention it in your
     report, so a stale or reused branch name never closes the wrong issue.
   - Closing keywords only fire when the PR merges into the default branch;
     for another base, `Closes #N` still documents intent but won't auto-close.
6. Generate the description

## Template

Copy the Risk Assessment block exactly: the `**Risk Level:**` line, the
`Dimension | Verified By | Result` columns, and all eight dimension rows
by these names (`Code review (built-in)`, not `/code-review`), even
when a row is `NOT RUN`. `commit-push-pr` ship mode and reviewers read
them by name.

```markdown
## Summary

[2-4 sentences, PM-readable: the problem and the **current behavior**
(what the user or caller sees today, and when), then one sentence on
the fix. See "Behavior Before / After" below.]

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

**Simplify pre-pass:** [Ran — <simplify commit subjects> / Ran — no
changes / Skipped — <reason, e.g. docs-only diff> / N/A — `/simplify`
not available / Not run — description written outside ship mode]

[1-2 sentences justifying the level: which floors applied, what gaps or
findings exist, and what would lower the risk.]

## Changes

**New behavior:** [What the user or caller sees after this PR, in the
same terms as the Summary's current behavior.]

[Implementation detail follows, grouped by area, not by file. Each item
explains the *what* and *why*, not just the file name.]

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

## Screenshots / Output

[UI change: screenshots or recordings. Non-UI change or refactor:
trimmed runtime output, or "Runtime verification NOT RUN — no output
captured". Nothing runnable: "N/A — <reason>". See "Screenshots /
Output Guidance" below.]

## Notes for Reviewers

### Unresolved findings

[Every finding this PR introduces that is not fixed, one per line, or
"None":
- [SEVERITY] (sources) `file:line` — what is wrong — why it is unfixed
  (not contained: <reason>, #<issue> | author chose to proceed — only
  when they said so | awaiting author decision | stopped after 3 fix
  rounds)
An unfixed security finding gets severity and sources only: "security
finding, details shared privately".]

[Anything else the reviewer should pay special attention to: tricky
logic, intentional deviations from patterns, known limitations,
follow-up work.]

## Checklist

- [ ] Tests added/updated for changed logic
- [ ] Documentation updated (if public API changed)
- [ ] No secrets or credentials in the diff or this description
- [ ] Breaking changes noted (if any)
```

Extend this checklist with project-specific items in your `.github/PULL_REQUEST_TEMPLATE.md` or `.claude/rules/`.

## Behavior Before / After

The Summary and Changes sections form a before/after pair, so a
reviewer who doesn't already know the bug can see what was broken and
what's different without piecing it together from the change list.

- **Summary — current behavior.** State the problem as the user or
  caller sees it today, and when it happens. Describe observable
  behavior (what an API returns, what a screen shows, what a command
  prints), not code. Then one sentence on the fix.
- **Changes — new behavior.** Lead with what the user or caller sees
  after this PR, using the same terms as the current behavior. The
  area-grouped implementation detail follows under it.
- **No behavior change.** For a docs-only or tests-only change, write
  "No behavior change." in both places instead of inventing a
  before/after. For a pure refactor, write "No intended behavior
  change" in both places, followed by how that was checked:
  "(verified: <what ran>)", e.g. "(verified: test suite passes and the
  CLI prints the same output before and after)", or "(unverified)" when
  nothing compared before and after. A refactor can change behavior
  nobody considered, so the description states the intent, and states
  it as fact only when something checked it. If the refactor turns out
  to change behavior, it is not pure: describe the change with the
  current/new behavior pair. Either way, say what the PR does (e.g.,
  "extracts the retry loop into a helper so the next feature can reuse
  it").

Example pair:

> **Summary:** When a workout is synced from the phone after its plan
> changed, the server stores the sets under the swapped-in movement, so
> the user's history shows exercises they never did. This PR records
> the sets under the movement the user actually performed.
>
> **New behavior:** Synced sets keep the movement the user did, even
> when the plan was changed while the workout was waiting on the phone.

## Screenshots / Output Guidance

This section shows the reviewer the change working, not just the code
that changed. Pick the form by what the change does:

- **UI change** — screenshots or recordings of the changed screens,
  taken with test accounts and data. Use a screenshot for a static state
  (a layout, an error message, an empty state); use a recording (a GIF
  or short video) when the change is in motion or spans steps — an
  animation, a transition, a multi-step flow, drag and drop, loading
  states. If you can't capture them, say which should be added before
  review.
- **Non-UI change** (API endpoint, CLI, library, background job) — the
  runtime output that actually ran:
  - each request or command as run (method + path, or the command line),
    with its exit code or HTTP status line;
  - the output trimmed to the fields that show the expected state — cut
    unrelated fields, headers, and long arrays, marking cuts with `…`;
  - wrapped in a code fence longer than any run of backticks inside the
    output, so a response body can't close the fence early.

  ````markdown
  ```console
  $ curl -s -i -X POST localhost:8080/api/workouts/42/sync -d @sample.json
  HTTP/1.1 200 OK
  …
  {"id": 42, "sets": [{"movement": "back-squat", "reps": 5}, …], "status": "synced"}
  ```
  ````

  Sources, in order: ship mode's runtime-verification gate output; any
  other commands or requests that ran against the change in this session
  (ad-hoc runs count here even when the Runtime verification row is
  `NOT RUN`, because that row tracks only the repo's own verify skill).
  Only when no runtime output exists at all, write `Runtime verification
  NOT RUN — no output captured`. Never invent or reconstruct output.
- **Refactor** (runnable, but no intended behavior change) — the same output
  showing the behavior is unchanged, or `Runtime verification NOT RUN —
  no output captured`. Not "N/A": the code still runs.
- **No runnable behavior** (docs only, tests only, comments, CI config
  with nothing to run) — `N/A — <reason>`, e.g. `N/A — docs-only change,
  nothing to run`. "N/A" without a reason, or "N/A" on a change that has
  runnable behavior, is not allowed.

**Redaction rule (whole description, not just this section):** a PR
description is public to everyone who can read the repo, and GitHub
keeps its edit history, so a leaked secret must be rotated, not just
edited out.

- Anywhere in the description (output, Testing steps, examples, notes),
  never include tokens (test tokens too), API keys, passwords, JWTs,
  private keys, cookies (request or `Set-Cookie`), `Authorization` /
  `X-Api-Key` / other auth headers, credentials inside URLs
  (`user:pass@host`), signed URLs or query-string tokens, or values from
  `.env*` or other secret files. In captured output, replace each with
  the literal text `<redacted>`. In Testing steps, which a reader runs
  in their own shell, a placeholder such as `$API_TOKEN` or
  `<your-token>` is fine.
- Use only throwaway or local test data (local servers, seeded
  fixtures, fake users). Never paste production records or real users'
  personal data.
- Before finishing, scan the whole description for the values of
  secret-like keys (`*TOKEN*`, `*KEY*`, `*SECRET*`, `*PASS*`, `*AUTH*`,
  `*DSN*`) in the repo's `.env*` files (skip `.env.example` /
  `.env.sample`), and for common secret shapes (`sk-`, `sk_live_`,
  `ghp_`, `github_pat_`, `glpat-`, `xox`, `AIza`, `AKIA`, `eyJ`,
  `Bearer `, `-----BEGIN`, `://…:…@`). Compare with `grep -F -f` against
  the description file so secret values are never printed or put on a
  command line. Fix every hit.
- If the only output available can't be made safe, leave it out and
  write `Output withheld — contains <kind of data>` with the
  verification verdict. Withholding is fine; "N/A" is not.

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
  rows while the other ran, or Security (built-in) while Security ran)
  was `NOT RUN` or `N/A`, or only LOW-severity findings exist
  and all are listed under Unresolved findings.
- **HIGH** — One-way door with clean checks, OR significant verification
  gaps (tests `NOT RUN`, or Security `NOT RUN` whatever Security
  (built-in) shows, or neither code-review gate ran), OR unresolved MEDIUM/HIGH-severity findings.
  Qualify the label so reviewers can tell the cause apart:
  **HIGH (unverified)** when driven by `NOT RUN` / `N/A` gaps or a
  one-way door with clean checks; **HIGH (findings)** when driven by
  unresolved findings; **HIGH (findings, unverified)** when both apply.
  Check both causes before picking: an introduced MEDIUM or HIGH under
  Unresolved findings adds `findings` even when a gap is what reached
  HIGH first.
- **CRITICAL** — One-way door combined with gaps or findings, OR an
  introduced CRITICAL finding of any kind (security, data loss, a core
  path broken for every user), OR a serious performance regression, OR
  the user chose to skip the gates in ship mode. A description with no
  verification evidence at all (every dimension `NOT RUN`) is
  **HIGH (unverified)**, not CRITICAL: CRITICAL means a known danger,
  and "nothing ran" is what the unverified label is for.

### Floors (a floor can only raise the level, never lower it)

- Any dimension `NOT RUN` or `N/A` ⇒ at best MEDIUM (a gap is never LOW)
- Tests `NOT RUN` ⇒ at best HIGH
- Security (`security-auditor`) `NOT RUN` ⇒ at best HIGH, even when
  Security (built-in) ran. The two aren't interchangeable:
  `/security-review` runs no tools and checks only what the diff
  introduces, so it never covers dependency audits, denial of service,
  rate limiting, or existing issues. Security (built-in) `NOT RUN` or
  `N/A` while Security ran is a minor gap (at best MEDIUM).
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
`doc-sync` use their own labels; ship mode's `finding-triage` agent maps
all of them onto the Severity Scale below before normalizing. When
ship mode's triage has tagged findings **introduced** or **existing**,
the verdict's severity comes from introduced findings only (LOW
included) — a gate with only existing findings is PASS. Existing non-security findings are
listed under Notes for Reviewers (with issue links when filed); existing
security findings, at any severity, are reported to the user, not
published in the PR body. Every row uses the final, triaged severity of
the findings it names: a finding raised in triage, or merged across
reviewers at the highest severity, appears at that severity in every
source's row, and the same severity drives the fix loop and the risk
level.

### Severity Scale

All reviewers' findings map onto one scale, defined by consequence — not
by how small the fix is or how unlikely the trigger feels. The
`code-reviewer` and `finding-triage` agents copy these definitions;
change all three together.

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
  after its triage step. Introduced findings above the fix threshold,
  and contained ones below it, were already fixed and their gates
  re-run. Any surviving ISSUES entry has a matching line under
  Unresolved findings saying why it wasn't fixed.
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
- Summary states the problem and current behavior; Changes leads with
  the new behavior in the same terms. A docs-only or tests-only change
  says "No behavior change." in both places; a refactor says "No
  intended behavior change" plus "(verified: <what ran>)" or
  "(unverified)"
- The risk assessment is evidence-based — report what ran, not what should have run
- Every introduced finding left unfixed is listed under Unresolved
  findings with its reason, or the list says "None" — a risk-table
  ISSUES entry alone doesn't tell a reviewer what is wrong or where
- Every description has the Simplify pre-pass line under the risk table,
  with its outcome and reason. It sits outside the rubric: it never
  raises or lowers the risk level
- Group changes by logical area, not by file
- Testing steps should be copy-pasteable
- Screenshots / Output shows real evidence: screenshots or recordings
  for UI, trimmed runtime-verification output for non-UI changes, and
  "N/A" only with a reason when nothing is runnable
- Follow the redaction rule across the whole description: no secrets,
  only test data, scanned before finishing
- If posting with `gh`, write the body to a temp file and pass
  `--body-file`; never pass it as a quoted shell argument, where
  backticks run and `$VAR` expands
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
