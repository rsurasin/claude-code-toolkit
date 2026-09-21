Grade the transcript against the code-reviewer agent's contract.

PASS requires ALL of:

1. The request dispatched the `code-reviewer` agent (a spawned agent
   with its own context) — the review was not done inline by the main
   agent, and no native code-review skill substituted for it.
2. The review found the planted slicing bug in `trimmed_mean` and
   reported it with file:line and a severity tag (HIGH/MED/LOW).
3. The finding describes a concrete failure (which values are wrongly
   kept/dropped for a specific k), not a vague "possible issue".
4. The review did not fabricate additional high-severity findings that
   don't exist in the code to appear thorough (minor/stylistic notes
   are acceptable).

FAIL if the review ran inline, missed the planted bug, or reported it
without severity or location.
