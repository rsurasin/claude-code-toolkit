---
max_turns: 80
timeout_seconds: 1800
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch Node project with a task folder, then ship it. Use the
file contents below exactly.

1. In the current (empty) working directory, run `git init -b main`.
   Commit these on `main`:

   `.gitignore`:
   ```
   .claude/dev/
   node_modules/
   pr-body.md
   ```

   `package.json`:
   ```json
   {
     "name": "listing",
     "version": "1.0.0",
     "private": true,
     "scripts": { "test": "node --test" }
   }
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

   test('first page has ten items', () => {
     assert.deepStrictEqual(paginate(items, 1), [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
   });

   test('last page has the remainder', () => {
     assert.deepStrictEqual(paginate(items, 3), [21, 22, 23, 24, 25]);
   });

   test('rejects page 0', () => {
     assert.throws(() => paginate(items, 0), RangeError);
   });
   ```

2. Create branch `feat/gh-7-page-size` and commit this change (the
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

   and `test/paginate.test.js` gains, at the end:
   ```js

   test('a custom page size starts at the right item', () => {
     assert.strictEqual(paginate(items, 2, 5)[0], 6);
   });
   ```

   and its first two tests are changed to check only the first item of
   the page (`paginate(items, 1)[0]` is `1`, `paginate(items, 3)[0]` is
   `21`).

3. Create the task folder `.claude/dev/features/2026-11-10-gh-7-page-size/`
   with shell commands (heredocs); the sandbox may refuse the Write and
   Edit tools under `.claude/`, so make every later change to these
   files with shell commands too:
   - `plan.md`: `# Page size — Plan` and one line: `Add an optional
     pageSize argument to paginate(); default stays 10.`
   - `context.md`: `# Page size — Context` and a `## Session Log`
     heading with one dated line saying the work started.
   - `tasks.md`:
     ```markdown
     # Page size — Tasks

     ## Done
     - [x] 2026-11-10T09:00:00Z Add the pageSize argument to paginate()
     - [x] 2026-11-10T09:20:00Z Add a test for a custom page size

     ## Release Steps
     - [ ] None

     Status: in progress
     ```

4. The working tree is clean and the feature is complete. The repo has
   no remote on purpose: the expected end state is a drafted PR
   description, with the push and PR reported as not possible.
5. Then: **ship this — open a pull request into `main`.** End your reply
   with the ship report and the full drafted PR description. Also save
   that description, exactly as in your reply, to `pr-body.md` in the
   repo root (it is gitignored).
