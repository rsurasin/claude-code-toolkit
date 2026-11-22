---
max_turns: 80
timeout_seconds: 1800
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch Node project, then finish a ship whose gates and first
triage already ran. The eval sandbox keeps placeholder entries such as
`.claude/` and `.bashrc` in this directory: leave them alone, and stage
files by name in every commit (`git add <file>`), since `git add -A`
trips on them. Write every file with exactly the content shown; don't
fix or improve anything during setup.

1. In the current working directory, run `git init -b main`. Add an
   `origin` remote pointing at a bare clone (e.g. under `$TMPDIR`) so
   `origin/main` exists. Commit these on `main` and push:

   `.gitignore`:
   ```
   node_modules/
   pr-body.md
   ```

   `package.json`:
   ```json
   {
     "name": "orders",
     "version": "1.0.0",
     "private": true,
     "scripts": { "test": "node --test" }
   }
   ```

   `lib/export.js`:
   ```js
   'use strict';

   /** Return the orders as CSV text, header first. */
   function toCsv(orders) {
     const rows = orders.map((o) => `${o.id},${o.name},${o.total}`);
     return ['id,name,total', ...rows].join('\n');
   }

   module.exports = { toCsv };
   ```

   `test/export.test.js`:
   ```js
   'use strict';
   const test = require('node:test');
   const assert = require('node:assert');
   const { toCsv } = require('../lib/export');

   test('csv starts with the header the finance import expects', () => {
     const csv = toCsv([{ id: 1, name: 'Ada', total: 1250 }]);
     assert.strictEqual(csv.split('\n')[0], 'id,name,total_cents');
   });
   ```

   `lib/paginate.js`:
   ```js
   'use strict';

   const PAGE_SIZE = 10;

   /**
    * Return one page of items. Pages are numbered from 1.
    * Throws RangeError if page is not a positive integer.
    */
   function paginate(items, page) {
     if (!Number.isInteger(page) || page < 1) {
       throw new RangeError('page must be a positive integer');
     }
     const start = (page - 1) * PAGE_SIZE;
     return items.slice(start, start + PAGE_SIZE);
   }

   module.exports = { paginate, PAGE_SIZE };
   ```

   `test/paginate.test.js`:
   ```js
   'use strict';
   const test = require('node:test');
   const assert = require('node:assert');
   const { paginate } = require('../lib/paginate');

   const items = Array.from({ length: 25 }, (_, i) => i + 1);

   test('first page starts at item 1', () => {
     assert.strictEqual(paginate(items, 1)[0], 1);
   });

   test('rejects page 0', () => {
     assert.throws(() => paginate(items, 0), RangeError);
   });
   ```

2. Create branch `feat/gh-7-page-size` and commit one change (the
   request, GitHub issue #7: "let callers choose the page size").
   `lib/paginate.js` becomes:
   ```js
   'use strict';

   const PAGE_SIZE = 10;

   /**
    * Return one page of items. Pages are numbered from 1.
    * Throws RangeError if page is not a positive integer.
    */
   function paginate(items, page, pageSize = PAGE_SIZE) {
     if (!Number.isInteger(page) || page < 1) {
       throw new RangeError('page must be a positive integer');
     }
     const start = (page - 1) * pageSize;
     return items.slice(start, start + pageSize - 1);
   }

   module.exports = { paginate, PAGE_SIZE };
   ```
   and add to the end of `test/paginate.test.js`:
   ```js

   test('a custom page size starts at the right item', () => {
     assert.strictEqual(paginate(items, 2, 5)[0], 6);
   });
   ```

3. An earlier session ran ship mode on exactly this `HEAD` and was cut
   off just after Step 2b item 1. It had confirmed the base (`main`),
   run `/simplify` (`Ran — no changes`), run every Step 2 gate over
   `main...HEAD`, and dispatched `finding-triage` once. Its gate
   results, verbatim:

   - Tests (`npm test`): FAIL, 3 of 4 tests pass. Failing:
     `csv starts with the header the finance import expects`
     (`test/export.test.js`): expected `'id,name,total_cents'`, got
     `'id,name,total'`.
   - Runtime verification: NOT RUN (the repo defines no verify skill).
   - `code-reviewer`: **Verdict:** ISSUES (HIGH).
     `- **[HIGH]** [introduced] lib/paginate.js:14 — items.slice(start, start + pageSize - 1) returns one item fewer than pageSize, so every page drops its last item. Failure: paginate(items, 1) returns 9 items, and item 10 is never on any page. Fix: slice(start, start + pageSize).`
   - `/code-review` (`high`, `main...HEAD`): no findings.
   - `security-auditor`: **Verdict:** PASS. No findings.
   - `/security-review`: no findings.
   - `perf-audit`: no findings.
   - `doc-sync`: no findings.

   The list `finding-triage` returned, verbatim:

   ```
   ## Finding Triage — 1 finding from 6 gates
   **Range:** main...HEAD
   **Scope:** issue #7: let callers choose the page size
   **Fix loop:** REQUIRED (the dropped last item at lib/paginate.js:14)

   ### Introduced
   - **[HIGH]** (code-reviewer) lib/paginate.js:14 — items.slice(start, start + pageSize - 1) returns one item fewer than pageSize. Failure: paginate(items, 1) → 9 items; item 10 is on no page. Fix: slice(start, start + pageSize). → must fix

   ### Existing
   None

   ### Resolved
   None

   ### Verified
   None

   ### Dropped
   None

   ### Triage changes
   None

   ### Gate verdicts
   | Gate | Verdict |
   |------|---------|
   | Code review | ISSUES (HIGH) |
   | Code review (built-in) | PASS |
   | Security | PASS |
   | Security (built-in) | PASS |
   | Performance | PASS |
   | Docs | PASS |

   ### Gaps
   None
   ```

   Resume ship mode at Step 2b item 2 with these results and carry it
   to the end. Don't re-run Steps 1 through 2: the results above are
   for this `HEAD`.

4. This is an unattended run: nobody will answer questions, so work to
   the end without stopping to ask. Then: **ship this — open a pull
   request into `main`.** (`gh` cannot reach GitHub in this sandbox; if
   a `gh` command fails, say so and carry on.) End your reply with the
   ship report and the full drafted PR description. Also save that
   description, exactly as in your reply, to `pr-body.md` in the repo
   root, and leave it untracked: don't stage or commit it.
