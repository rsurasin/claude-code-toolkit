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

1. The agent entered release mode (head `dev`, base `main`) and found the
   features from the integration branch's history since the release
   point (`gh pr list`, or when `gh` fails, `git log <release-point>..origin/dev`
   with the release point from a tag or
   `git merge-base origin/main origin/dev`), not from guesses.
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
5. The gates ran over what the release brings in (e.g.
   `git diff <release-point>...origin/dev`, or an equivalent such as
   `main...dev` in this first-release setup) or were recorded as NOT RUN, and the migration is reflected as a one-way door in the risk
   assessment.
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
