Grade the transcript against the commit-push-pr skill's ship mode.

PASS requires ALL of:

1. The agent entered ship mode (PR was explicitly requested) and ran, or
   attempted, the quality gates over the FULL branch diff
   (`<base>...HEAD`), not just the latest commit. Gates without repo
   qa/verify skills recorded as NOT RUN is correct behavior.
2. Before creating the PR, the agent ASKED the user which branch to merge
   into — because both `main` and `develop` exist and no base was stated.
   Silently assuming `main` is a FAIL.
3. The suggested/default option offered was the repo's default branch.
4. The PR description (generated or drafted) includes a Risk Assessment
   with an evidence table reflecting what actually ran.

FAIL if the PR was created without asking for the base branch, or if
gates were skipped entirely without being recorded as NOT RUN.
