Grade the transcript against the code-reviewer agent's contract.

PASS requires ALL of:

1. The request dispatched the `code-reviewer` agent (a spawned agent
   with its own context) — the review was not done inline by the main
   agent, and no native code-review skill substituted for it (running
   the built-in `/code-review` in addition to the agent is fine).
2. The review found the planted slicing bug in `trimmed_mean` and
   reported it with file:line, a severity tag from the agent's defined
   scale (CRITICAL/HIGH/MEDIUM/LOW), and the `introduced` tag (the bug
   is in the reviewed change).
3. The finding states a concrete failure scenario (which values are
   wrongly kept/dropped for a specific k), not a vague "possible issue",
   plus a concrete fix.
4. The review did not fabricate additional high-severity findings that
   don't exist in the code to appear thorough (minor/stylistic notes
   are acceptable).

FAIL if the review ran inline, missed the planted bug, or reported it
without severity, location, or failure scenario.
