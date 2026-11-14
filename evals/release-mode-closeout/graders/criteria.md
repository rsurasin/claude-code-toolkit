---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the commit-push-pr skill's release mode
(opening phase) and the dev-tasks release closeout.

Note: the eval sandbox may refuse writes under `.claude/`, in the setup or
later. If it did, that is the environment, not a failure — grade the
writes the agent attempted (the commands in the trace). Writing the files
with a shell heredoc instead of the Write tool is fine.

PASS requires ALL of:

1. The agent entered release mode (head `dev`, base `main`). (A
   separate grader checks that it read the release point from the
   repository's history; the trace you see is cut down, so do not fail
   the run because that command is not visible.)
2. It mapped both PRs to their task folders by the `gh-<N>` in the branch
   or folder name, or by the closed issue number.
3. The release PR description (created or drafted) contains:
   - a feature list naming PR #41 (password reset, closes #31) and
     PR #43 (sets cap, closes #36);
   - a release steps section with BOTH steps: deploying together with
     acme/web#56, and applying migration `0042_cap_sets`;
   - `Closes #31` and `Closes #36`.
4. The description, PR title, and any commit message carry zero trace of
   dev-tasks: no `.claude/dev/` paths, no folder names (such as
   `2026-10-04-gh-31-password-reset`), no dev-tasks task IDs, no
   bookkeeping mentions.
5. The risk assessment has a result for every gate (ran, or recorded
   NOT RUN), and reflects the migration as a one-way door. (Separate
   graders check that the review agents were dispatched; do not fail
   the run because a gate's command is not visible in the trace.)
6. Neither folder was marked `released`: both still end in
   `merged to dev (...), not released`, because the release PR has not
   merged.
7. After the step-5 request, no commit was made on `dev` or `main`
   (gate fixes, if any, go to a new branch with the user's approval). If
   `gh pr create` was attempted, the body went through `--body-file`, not
   an inline `--body` string.

FAIL if either release step is missing, a `Closes #N` line is missing, any
`.claude/dev/` path or folder name appears in the PR text, or a folder was
marked released before the merge.
