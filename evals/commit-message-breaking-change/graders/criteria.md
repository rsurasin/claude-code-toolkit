Grade the transcript against the commit-message skill's
breaking-change conventions.

PASS requires ALL of:

1. The message uses the breaking marker: `<type>(<scope>)!:` (e.g.,
   `refactor(api)!:` or `feat(api)!:`).
2. The body includes a `BREAKING CHANGE:` footer describing the renamed
   function and removed parameter.
3. Subject line rules hold: imperative mood, no trailing period, max 72
   characters, lowercase first letter after the colon.
4. The body explains WHAT changed and WHY, not a restatement of the
   diff.

FAIL if the `!` marker or the `BREAKING CHANGE:` footer is missing, or
the subject violates the format rules.
