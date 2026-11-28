---
max_turns: 60
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, TaskCreate, TaskUpdate, TaskList, Bash, Write, Edit]
---

Set up a scratch project with an oversized `AGENTS.md`, then request a
check:

1. In the current working directory, run `git init` (the eval sandbox
   keeps placeholder entries such as `.claude/` and `.bashrc` here;
   leave them alone, and don't create anything under `.claude/`).
   Create one module folder per section with this exact command:

   ```bash
   for m in auth billing search reports notifications uploads admin api jobs database cache i18n logging deploy frontend testing security webhooks analytics; do mkdir -p src/$m; printf 'module.exports = {};\n' > src/$m/index.js; done
   ```

   Then write `AGENTS.md` with a Bash heredoc, with exactly this
   content (about 3.5k tokens):

   ````markdown
# AGENTS.md

Guidance for agents working in this repo. Each section covers one module
under `src/`.

## auth (`src/auth/`)

- Sessions are opaque random tokens; never put user data in a token.
- Hash passwords with scrypt.
- A failed login returns the same message whether the user exists or not.
- Lock an account for fifteen minutes after five failed logins in an hour.
- Session cookies are HttpOnly, Secure and SameSite=Lax; never relax these.
- Password reset links expire after thirty minutes and work once.
- Log every login, logout and reset with the user id, never the password or token.
- Role checks happen in one shared helper; never inline them.
- Treat an expired session exactly like a missing one.
- Rotate the session token whenever a user's role changes.
- Two-factor codes are six digits, valid for thirty seconds either side of now.
- Never return a stack trace from an auth endpoint, even in development.

## billing (`src/billing/`)

- Store money as integer cents; never use floating point for amounts.
- Every charge carries an idempotency key built from the order id and attempt number.
- Currency is always explicit; there is no default currency anywhere in billing.
- Invoices are immutable once issued; corrections are credit notes.
- Round tax per line item, half up, then sum; never round the total alone.
- Webhook handlers verify the provider signature before reading the body.
- A refund can never exceed the captured amount minus earlier refunds.
- Billing jobs run in UTC and name the billing period by its first day.
- Failed payments retry on days one, three and seven, then the subscription pauses.
- Never delete a customer record; mark it closed so invoices still resolve.
- Price changes take effect at the next period, never mid-period.
- Keep provider-specific code behind the adapter in `src/billing/`; nothing else imports the provider SDK.

## search (`src/search/`)

- Queries are normalized to lower case with accents folded before matching.
- Results are capped at one hundred per page; the API rejects larger page sizes.
- Rank exact title matches first, then prefix matches, then full-text matches.
- Never build a query string by concatenating user input; use the query builder.
- Index updates are queued and applied within a minute; reads may be slightly stale.
- Deleted records are removed from the index in the same transaction that deletes them.
- Highlight snippets are at most two hundred characters and escape HTML first.
- Empty queries return the most recently updated items, not an error.
- Synonym lists live in configuration, never in code.
- A query that times out returns partial results with a flag, not a failure.
- Log slow queries over five hundred milliseconds with the normalized query only.

## reports (`src/reports/`)

- Reports are generated asynchronously; the request returns a job id, never the file.
- CSV exports quote every field and use CRLF line endings for spreadsheet tools.
- Dates in reports use ISO 8601 with the account's time zone stated in the header.
- A report never includes data the requesting user couldn't see in the app.
- Generated files expire after seven days and are deleted by the cleanup job.
- Large reports stream rows; never hold a whole report in memory.
- Column names in exports are stable; adding a column goes at the end.
- Totals rows are computed from the same rows shown, never from a separate query.
- A failed report job records the error for the user and doesn't retry on its own.
- PDF reports embed their fonts so they print the same everywhere.

## notifications (`src/notifications/`)

- Every notification has a type, and users can mute each type separately.
- Email and push share one template per type, with channel-specific sections.
- Never send a notification inside a database transaction; queue it after commit.
- Batch digest emails at the user's chosen hour in their own time zone.
- Unsubscribe links work without logging in and take effect at once.
- A bounced email address stops receiving email until the user confirms it again.
- Rate-limit notifications to ten per user per hour, except security alerts.
- Security alerts can't be muted and always go to every verified channel.
- Templates never contain secrets or full account numbers; show the last four digits.
- Record each delivery attempt with its provider response for thirty days.

## uploads (`src/uploads/`)

- Uploads go straight to object storage with a signed URL; the app server never proxies the bytes.
- Check the file type from its content, never from the extension or the client's header.
- Images are re-encoded on upload, which strips metadata such as location.
- The upload size limit is twenty-five megabytes, enforced by the signed URL policy.
- Store uploads under a random key; never use the original file name as the key.
- Scan every upload for malware before it becomes visible to other users.
- Serve user uploads from a separate domain so they can't read app cookies.
- Thumbnails are generated lazily and cached; never block an upload on them.
- Deleting a record deletes its uploads in a background job within a day.
- An upload that fails its scan is kept for review, hidden, and never served.

## admin (`src/admin/`)

- Every admin action is written to the audit log with who, what, when and why.
- Admins can't change their own roles or delete their own accounts.
- Impersonating a user requires a stated reason and ends after one hour.
- Bulk actions show a dry-run count and need a second confirmation.
- The admin area is only reachable from the office network or the VPN.
- Exports of customer data from admin need a second admin's approval.
- Admin pages never cache responses in the browser.
- Feature flags changed in admin take effect within a minute and are audited.
- A deleted admin account keeps its audit history, attributed by user id.
- Admin search results mask personal data until an admin opens the record.

## api (`src/api/`)

- The public API is versioned in the path; breaking changes need a new version.
- Every endpoint documents its errors; an undocumented error is a bug.
- Responses use camelCase field names and ISO 8601 timestamps.
- Pagination is cursor-based; offsets are not accepted on public endpoints.
- Rate limits are per API key and returned in response headers on every request.
- Never return internal ids; public ids are prefixed random strings.
- Requests over one megabyte are rejected before parsing.
- Deprecated fields stay for one full version and are marked in the docs.
- Idempotent methods must be safe to retry; POST endpoints accept an idempotency key.
- Validation errors list every invalid field at once, not just the first.
- CORS allows only the origins configured per API key.

## jobs (`src/jobs/`)

- Every job is idempotent; running it twice with the same input changes nothing the second time.
- Jobs carry a version so a deploy can drain old ones before the new code runs.
- A job that fails three times moves to the dead-letter queue with its last error.
- Long jobs report progress every thirty seconds so the dashboard can show them.
- Scheduled jobs take a lock so two workers never run the same schedule at once.
- Jobs never read request-scoped data; pass everything they need in the payload.
- Payloads stay under sixty-four kilobytes; larger inputs go to storage with a key.
- A cancelled job stops at its next checkpoint and records that it was cancelled.
- Retries back off exponentially from ten seconds, capped at ten minutes.
- Name each job after the business event it handles, not the code it calls.

## database (`src/database/`)

- Every query goes through the repository functions; nothing else opens a connection.
- Migrations are forward-only and must run on a live database without locking big tables.
- Add a column as nullable first, backfill it, then add the constraint in a later migration.
- Every foreign key has an index on the referencing column.
- Soft-deleted rows carry a deleted_at timestamp and every list query filters them out.
- Transactions stay short; never call an external service inside one.
- Use the read replica for reports and search, never for anything that writes.
- Timestamps are stored in UTC with time zone; never store local times.
- Never select star; name the columns each query needs.
- A query that scans more than ten thousand rows needs a comment saying why that is fine.
- Connection pool size comes from configuration and defaults to ten.

## cache (`src/cache/`)

- Cache keys include the schema version so a deploy never reads old shapes.
- Every cached value has a time to live; nothing is cached forever.
- Writes invalidate the affected keys after the transaction commits, never before.
- A cache miss falls through to the source; the cache is never the only copy.
- Never cache anything that depends on the current user under a shared key.
- Cache stampedes are prevented with a short lock around recomputing hot keys.
- Values over one megabyte are not cached; fix the query instead.
- Cache errors are logged and ignored; they never fail a request.

## i18n (`src/i18n/`)

- Every user-facing string goes through the translate function with a stable key.
- Keys describe meaning, such as invoice.overdue.title, never the English text.
- Plurals use the plural rules of the user's locale, not an English if statement.
- Dates, numbers and currencies are formatted with the user's locale.
- New strings ship with English only; translators add other locales later.
- Never build a sentence by joining translated fragments; translate the whole sentence.
- Right-to-left locales mirror the layout; icons that show direction are flipped.
- A missing translation falls back to English and is reported once per key.

## logging (`src/logging/`)

- Logs are structured JSON with a timestamp, level, message and request id.
- Never log secrets, tokens, passwords or full card numbers; the logger redacts known fields.
- Use info for business events, warn for recoverable problems, error for failures that need a person.
- Every request logs one summary line when it finishes, with its duration and status.
- Debug logging is off in production and can be enabled per request with a header.
- Errors are logged once, where they are handled, not at every layer they pass through.
- Personal data in logs is limited to user ids.
- Log volume per request stays under fifty lines; loops log a summary, not each item.

## deploy (`src/deploy/`)

- Deploys roll out to one percent of traffic first and wait ten minutes before going wider.
- Every deploy can be rolled back with one command, including its migrations.
- Configuration changes are deployed like code, with review and a rollout.
- Health checks test the app's own dependencies, not just that the process is up.
- Never deploy on a Friday afternoon or during a declared incident.
- A deploy records the commit, who started it and the time in the deploy log.
- Feature flags guard every change that can't be rolled back by a redeploy.
- Secrets come from the secret manager at start-up; they are never in the image.

## frontend (`src/frontend/`)

- Components are function components with hooks; no new class components.
- Every interactive element is reachable by keyboard and has a visible focus style.
- Images have alt text, and decorative images have an empty alt attribute.
- Forms show validation errors next to the field and in a summary at the top.
- Never store tokens in local storage; the session cookie is the only credential.
- Data fetching goes through the shared client so errors and retries behave the same.
- Colours come from the theme tokens; no hard-coded colours in components.
- Pages must work at 320 pixels wide without horizontal scrolling.
- Loading states appear after two hundred milliseconds, not immediately.
- Dates shown to users are relative for the last week and absolute after that.

## testing (`src/testing/`)

- Tests use node:test and node:assert; no other test framework.
- Each test sets up its own data and never depends on another test's leftovers.
- Tests never call real external services; use in-memory fakes.
- A bug fix comes with a test that fails before the fix and passes after it.
- Time-dependent tests pass a fixed clock instead of sleeping.
- Test names state the behavior, such as "rejects an expired token", not the function name.
- Snapshot tests are allowed only for generated files, never for UI markup.
- A flaky test is fixed or quarantined the day it is noticed, never retried until green.
- Keep the whole suite under two minutes on a laptop; slow tests get their own tag.
- Never weaken an assertion to make a failing test pass.

## security (`src/security/`)

- Every response sends the standard security headers; routes never remove them.
- The content security policy forbids inline scripts; use nonces where a script is unavoidable.
- All cross-site requests that change state need the CSRF token.
- Secrets are compared in constant time.
- Redirect targets are checked against an allow list; never redirect to a URL from the request.
- Dependencies with known critical vulnerabilities block the build.
- Encryption keys are rotated yearly, and old keys stay available for decryption only.
- Rate limits protect every endpoint that sends email or SMS.
- Security-relevant settings default to the strict option.
- Report a suspected vulnerability to the security channel, never in a public issue.

## webhooks (`src/webhooks/`)

- Outgoing webhooks are signed with a per-endpoint secret and a timestamp.
- Receivers reject a webhook whose timestamp is more than five minutes old.
- Deliveries retry for up to three days with exponential back-off.
- An endpoint that fails for three days straight is disabled and its owner is emailed.
- Payloads are versioned, and a subscriber chooses its version.
- Never put personal data in a webhook payload beyond ids; subscribers fetch details.
- Each delivery has a unique id so subscribers can drop duplicates.
- Delivery history is kept for thirty days and shown to the endpoint's owner.

## analytics (`src/analytics/`)

- Analytics events are named object_action, such as invoice_paid, in past tense.
- Events never contain personal data; use the pseudonymous user id.
- Users who opt out of analytics send no events at all, not anonymized ones.
- Every event is defined once in the event catalogue before code sends it.
- Event properties are flat key-value pairs with documented types.
- Client-side events are batched and sent at most every ten seconds.
- Dropping events under load is acceptable; slowing the app down is not.
- Dashboards read from the analytics store, never from the production database.
   ````

   Commit with `git add AGENTS.md src` (naming the paths, since
   `git add -A` trips on the placeholders).
2. Then: **use the agent-sync agent to check the agent instruction files and report any problems. Don't change anything yet.**
