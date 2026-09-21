---
name: sharpen
description: >
  Encode a mistake or inefficiency into a permanent harness improvement —
  a test, lint rule, or CLAUDE.md / .claude/rules/ entry that prevents
  recurrence. Use proactively whenever a sharpening moment appears: the
  user corrects your work or re-explains something they've said before,
  you discover your own mistake, or you hit the same friction twice.
  Always propose the improvement and get explicit user consent before
  modifying any harness file. This is the sword-sharpening loop — each
  session makes the next one more capable.
---

# Sharpen — Mistake-to-Harness Loop

A mistake or inefficiency was just observed. Your job is to ensure this
specific class of problem never happens again by creating a permanent
artifact — with the user's consent.

## When to Invoke

Invoke this skill proactively; don't wait for `/sharpen`.

**User-detected signals:**
- The user corrects your output ("no, do it this way", "that's wrong")
- The user re-explains a preference or constraint they've stated before
- The user explicitly runs `/sharpen`

**Agent-detected signals:**
- You discover a mistake in your own earlier work
- You redo work because an assumption turned out to be wrong
- You hit the same friction twice in a session (missing context, a
  command you had to rediscover, a convention you had to re-infer)

One-off trivia doesn't qualify — sharpen when the *class* of problem
would plausibly recur in a future session.

## Consent Gate (never skip)

The harness belongs to the user. **Never modify a harness file —
CLAUDE.md, `.claude/rules/`, lint config, or a guard test — without
explicit consent given in this conversation.**

Before creating anything, propose:

```
## Sharpening opportunity

**Observed:** [the mistake or inefficiency, and who caught it]
**Class:** [the general class of problem this belongs to]
**Proposed artifact:** [what would be created/changed, and where]
**Draft:** [the actual rule text / test outline]

Add this to the harness? (yes / edit / no)
```

If the user declines, drop it — do not re-propose the same rule later in
the session. If they edit, apply their version. Prior consent for a
different rule does not carry over; each artifact gets its own approval.

## Process (after consent)

### Step 1: Identify the Mistake

Confirm from context (ask only if genuinely unclear):
- **What happened?** (the incorrect behavior or inefficiency)
- **What should have happened?** (the expected behavior)
- **Where?** (which file/function/pattern)

### Step 2: Classify the Fix Type

Determine which artifact will prevent recurrence:

| Mistake Type | Artifact to Create |
|---|---|
| Wrong pattern used | CLAUDE.md rule or `.claude/rules/` entry |
| Type error / wrong signature | Stricter type, type guard, or schema validation |
| Logic bug | Unit test covering the edge case |
| Style / convention violation | Lint rule or CLAUDE.md convention entry |
| Architecture violation | `.claude/rules/` entry with explanation |
| Wrong dependency usage | CLAUDE.md entry with correct usage example |
| Repeated prompting issue | `.claude/rules/` entry with the correction |

### Step 3: Create the Artifact

**If adding a CLAUDE.md rule:**
- Open CLAUDE.md
- Find the appropriate section (or create one)
- Add a concise rule: what NOT to do, what TO do instead, and why
- Keep it under 3 lines — if it needs more, put it in a rules file

**If adding a rules file entry:**
- Check if a relevant `.claude/rules/*.md` file exists
- If so, add the rule to the existing file
- If not, create a new rules file for the domain
- Include: the wrong pattern, the correct pattern, and why

**If adding a test:**
- Create a test that would have caught this specific mistake
- Place it in the appropriate test directory
- The test should fail if the mistake is reintroduced
- Run the test to confirm it passes with the current (fixed) code

**If adding a lint rule or type guard:**
- Add the rule/guard
- Verify it would catch the original mistake
- Ensure it doesn't produce false positives on existing code

### Step 4: Verify

- Re-read the artifact you created
- Confirm it's specific enough to catch this mistake
- Confirm it's general enough to catch the same *class* of mistake
- Run any tests to confirm they pass

### Step 5: Report

```
## Sharpened

**Mistake:** [brief description]
**Prevention:** [what was added and where]
**Scope:** [will this catch similar mistakes? how broad is the protection?]
```

## Rules

- The consent gate is absolute — no harness file changes without explicit
  approval in this conversation, even when you detected the mistake
  yourself
- Every artifact should be the minimum needed to prevent the mistake
- Prefer tests over CLAUDE.md rules when possible — tests are enforceable
- Prefer CLAUDE.md rules over verbal corrections — rules persist across
  sessions
- Never add a rule that contradicts an existing rule — check first
    - If there is a contradiction, flag it so the user can review
- Before proposing a new artifact, check if an existing `.claude/rules/`
  entry or CLAUDE.md rule already covers this class of mistake
- If the same class of mistake has been sharpened before, strengthen the
  existing artifact rather than adding a duplicate
- Sharpening should take < 5 minutes; batch multiple observations into
  one proposal rather than interrupting the user repeatedly
