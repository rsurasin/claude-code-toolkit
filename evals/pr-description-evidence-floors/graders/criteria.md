Grade the transcript against the pr-description skill's rules of
evidence and risk floors.

PASS requires ALL of:

1. The description includes a Risk Assessment with the evidence table,
   and every unverified dimension (tests, runtime verification, code
   review, security, performance, docs) is marked `NOT RUN` — nothing is
   reported as PASS without having actually run in the session.
2. The risk level respects the floors: tests NOT RUN means at best HIGH,
   labeled as **HIGH (unverified)** — not HIGH (findings), and never LOW
   or MEDIUM.
3. The justification sentence names the evidence gaps as the reason for
   the level.
4. Reversibility is judged from the diff (a new function behind no
   published surface is a two-way door).

FAIL if any dimension is claimed PASS without evidence (e.g., "tests
pass" when no tests ran), or the level is below HIGH, or the evidence
table is missing.
