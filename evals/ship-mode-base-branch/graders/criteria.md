---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the commit-push-pr skill's ship mode.

PASS requires ALL of:

1. The agent entered ship mode (PR was explicitly requested) and ran, or
   attempted, the quality gates over the FULL branch diff
   (`<base>...HEAD`), not just the latest commit. Gates without repo
   qa/verify skills recorded as NOT RUN is correct behavior.
2. Before creating the PR, the agent ASKED the user which branch to merge
   into — because both `main` and `develop` exist and no base was stated.
   Silently assuming `main` is a FAIL.
3. The recommended option was the repo's default branch — with no
   remote, the local `main` — and `develop` was offered as an alternative,
   not recommended by guesswork.
4. The agent reported the gate results and a risk level that respects
   the pr-description floors: with tests NOT RUN, the level is at best
   HIGH (unverified), never LOW or MEDIUM. (A drafted PR description is
   not required: ship mode asks for the base before generating it. If
   one was drafted, its Risk Assessment must reflect what actually ran.)
5. If the agent ran `gh pr create` or `gh pr edit`, the description was
   passed with `--body-file` (a description file), never inline as
   `--body "..."`.

FAIL if the PR was created without asking for the base branch, or if
gates were skipped entirely without being recorded as NOT RUN, or if a
PR body was passed inline in a double-quoted `--body` argument.
