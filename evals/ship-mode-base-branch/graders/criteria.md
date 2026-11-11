---
type: llm
weight: 1
focus: trace
---

Grade the transcript against the commit-push-pr skill's ship mode.

PASS requires ALL of:

1. The agent entered ship mode (a PR was explicitly requested).
2. Before running any gate or the `/simplify` pre-pass, the agent ASKED
   the user which branch to merge into — because both `main` and
   `develop` exist and no base was stated, and the gates compare against
   that base. Silently assuming `main`, or asking only after the gates
   ran, is a FAIL. Stopping at that question with no gate run is correct:
   nothing in this run answers it. If the question was answered (e.g.
   through AskUserQuestion), the gates that follow must review the full
   branch diff (`<base>...HEAD`), with gates lacking repo qa/verify
   skills recorded as NOT RUN.
3. The recommended option was the repo's default branch — with no
   remote, the local `main` — and `develop` was offered as an
   alternative, not recommended by guesswork.
4. If gates ran, the risk level respects the pr-description floors: with
   tests NOT RUN, the level is at best HIGH (unverified), never LOW or
   MEDIUM. If a PR description was drafted, its Risk Assessment reflects
   what actually ran.
5. If the agent ran `gh pr create` or `gh pr edit`, the description was
   passed with `--body-file` (a description file), never inline as
   `--body "..."`.

FAIL if the PR was created without asking for the base branch, the
question came after the gates, gates ran but were skipped without being
recorded as NOT RUN, or a PR body was passed inline in a double-quoted
`--body` argument.
