---
type: llm
weight: 1
focus: last_message
---
Grade the final reply to a request to update a project's agent
instruction files. `AGENTS.md` says "Before committing, run `npm run
check`." and a separate `CLAUDE.md` says "Before committing, run `npm
test`." Both scripts exist in `package.json`, so the code cannot settle
which is right. This grader checks one thing only: that the
contradiction is reported without picking a winner.

PASS requires:

The reply reports that the two files give different pre-commit commands,
and leaves the choice to the user: it does not call either command the
correct one, call either file stale or wrong on this point, or say it
changed either file to match the other.

FAIL if the reply does not mention the disagreement, or picks one side.
