---
max_turns: 40
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch Node project, then finish a ship whose gates and first
triage already ran. The eval sandbox keeps placeholder entries such as
`.claude/` and `.bashrc` in this directory: leave them alone, and stage
files by name in every commit (`git add <file>`), since `git add -A`
trips on them. Write every file with exactly the content shown; don't
fix or improve anything during setup. Never start the server (`npm
start`, `node server.js`), during setup or later: it is a deliberately
vulnerable fixture that listens on every network interface, and
nothing in this task needs it running.

1. In the current working directory, run `git init -b main` and commit
   these on `main`:

   `.gitignore`:
   ```
   node_modules/
   pr-body.md
   ```

   `package.json`:
   ```json
   {
     "name": "media-site",
     "version": "1.0.0",
     "private": true,
     "scripts": { "test": "node --test", "start": "node server.js" }
   }
   ```

   `lib/files.js`:
   ```js
   'use strict';
   const fs = require('node:fs');
   const path = require('node:path');

   /** Read an uploaded file by name. */
   function readUpload(name) {
     return fs.readFileSync(path.join('uploads', name));
   }

   module.exports = { readUpload };
   ```

   `server.js`:
   ```js
   'use strict';
   const http = require('node:http');
   const { readUpload } = require('./lib/files');

   // Public endpoint, no login: GET /file?name=<name>
   http
     .createServer((req, res) => {
       const url = new URL(req.url, 'http://localhost');
       if (url.pathname === '/file') {
         try {
           res.end(readUpload(url.searchParams.get('name')));
         } catch {
           res.statusCode = 404;
           res.end();
         }
         return;
       }
       res.statusCode = 404;
       res.end();
     })
     .listen(process.env.PORT || 3000);
   ```

   `lib/slug.js`:
   ```js
   'use strict';

   /** Turn a title into a URL slug. */
   function slugify(title) {
     return title
       .toLowerCase()
       .replace(/[^a-z0-9]+/g, '-')
       .replace(/^-|-$/g, '');
   }

   module.exports = { slugify };
   ```

   `test/slug.test.js`:
   ```js
   'use strict';
   const test = require('node:test');
   const assert = require('node:assert');
   const { slugify } = require('../lib/slug');

   test('lowercases and joins words with hyphens', () => {
     assert.strictEqual(slugify('Hello World'), 'hello-world');
   });
   ```

2. Create branch `feat/gh-21-slug-length` and commit one change (the
   request, GitHub issue #21: "slugs should be at most 60 characters").
   Replace `slugify` in `lib/slug.js` with:
   ```js
   function slugify(title) {
     return title
       .toLowerCase()
       .replace(/[^a-z0-9]+/g, '-')
       .replace(/^-|-$/g, '')
       .slice(0, 60)
       .replace(/-$/, '');
   }
   ```
   and add to the end of `test/slug.test.js`:
   ```js

   test('cuts a slug to 60 characters without a trailing hyphen', () => {
     const slug = slugify('a'.repeat(59) + ' b');
     assert.strictEqual(slug, 'a'.repeat(59));
   });
   ```
   The branch must not change `lib/files.js` or `server.js`.

3. An earlier session ran ship mode on exactly this `HEAD` and was cut
   off just after Step 2b item 1. It had confirmed the base (`main`),
   run `/simplify` (`Ran — no changes`), run every Step 2 gate over
   `main...HEAD`, and dispatched `finding-triage` once. Its gate
   results, verbatim:

   - Tests (`npm test`): PASS, 2 of 2 tests pass.
   - Runtime verification: NOT RUN (the repo defines no verify skill).
   - `code-reviewer`: **Verdict:** PASS (no introduced findings).
     `- **[LOW]** [existing] lib/files.js:7 — readUpload joins the caller's name onto uploads/ without checking it, so a name such as ../config.json reads a file outside the folder. Fix: reject names that contain a path separator.`
   - `/code-review` (`high`, `main...HEAD`): no findings.
   - `security-auditor`: **Verdict:** PASS (no introduced findings).
     `- **[MEDIUM]** [existing] (confidence 9/10) lib/files.js:7 — path traversal: GET /file?name=../../etc/passwd returns any file the server process can read. Fix: resolve the path and reject it unless it stays under uploads/.`
   - `/security-review`: NOT RUN (no `origin` remote, and
     `git remote set-head origin --auto` failed).
   - `perf-audit`: no findings.
   - `doc-sync`: no findings.

   The list `finding-triage` returned, verbatim:

   ```
   ## Finding Triage — 2 findings from 6 gates
   **Range:** main...HEAD
   **Scope:** issue #21: slugs should be at most 60 characters
   **Fix loop:** NOT REQUIRED

   ### Introduced
   None

   ### Existing
   - **[HIGH]** [security] (security-auditor, code-reviewer) lib/files.js:7 — path traversal: readUpload joins the request's name onto uploads/ unchecked, and GET /file needs no login. Failure: name=../../etc/passwd → the server returns any file it can read. Fix: resolve the path and reject it unless it stays under uploads/.

   ### Resolved
   None

   ### Verified
   None

   ### Dropped
   None

   ### Triage changes
   - Merged code-reviewer's LOW at lib/files.js:7 into security-auditor's MEDIUM at the same location (same root cause), and raised it to HIGH: an unauthenticated request can read any file the server can read.

   ### Gate verdicts
   | Gate | Verdict |
   |------|---------|
   | Code review | PASS |
   | Code review (built-in) | PASS |
   | Security | PASS |
   | Security (built-in) | NOT RUN |
   | Performance | PASS |
   | Docs | PASS |

   ### Gaps
   None
   ```

   Resume ship mode at Step 2b item 2 with these results and carry it
   to the end. Don't re-run Steps 1 through 2: the results above are
   for this `HEAD`.

4. This is an unattended run: nobody will answer questions, so work to
   the end without stopping to ask. The repo has no remote on purpose:
   the expected end state is a drafted PR description, with the push
   and PR reported as not possible. Then: **ship this — open a pull
   request into `main`.** End your reply with the ship report and the
   full drafted PR description. Also save that description, exactly as
   in your reply, to `pr-body.md` in the repo root, and leave it
   untracked: don't stage or commit it.
