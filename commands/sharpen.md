---
description: >
  Encode a mistake into a permanent improvement. After Claude makes an error,
  run /sharpen to create a test, lint rule, or CLAUDE.md entry that prevents
  it from recurring. This is the sword-sharpening loop — each session makes
  the next one more capable.
---

# Sharpen — Mistake-to-Harness Loop

You just observed Claude make a mistake. Your job is to ensure this specific
class of mistake never happens again by creating a permanent artifact.

## Process

### Step 1: Identify the Mistake

Ask the user (and infer from context):
- **What happened?** (the incorrect behavior)
- **What should have happened?** (the expected behavior)
- **Where?** (which file/function/pattern)
- **ALWAYS** get the user's permission to add the new rule

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

- Before adding a rule or test, get the user's permission
- Every artifact should be the minimum needed to prevent the mistake
- Prefer tests over CLAUDE.md rules when possible — tests are enforceable
- Prefer CLAUDE.md rules over verbal corrections — rules persist across sessions
- Never add a rule that contradicts an existing rule — check first
    - If there is a contradiction, flag it so the user can review
- Before creating a new artifact, check if an existing `.claude/rules/`
  entry or CLAUDE.md rule already covers this class of mistake
- If the same class of mistake has been sharpened before, strengthen the
  existing artifact rather than adding a duplicate
- This command should take < 5 minutes
