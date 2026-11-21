---
max_turns: 80
timeout_seconds: 1800
allowed_tools: [Read, Glob, Grep, Skill, Agent, AskUserQuestion, Bash, Write, Edit]
---

Set up a scratch Node project, then ship a small feature unattended.
Use the file contents below exactly.

1. In the current (empty) working directory, run `git init -b main`.
   Add an `origin` remote pointing at a bare clone (e.g. under
   `$TMPDIR`). Commit these on `main`, push, create a long-lived
   `develop` branch from it and push that too, then run
   `git remote set-head origin main` so `origin/HEAD` points at
   `main`:

   `.gitignore`:
   ```
   node_modules/
   pr-body.md
   ```

   `package.json`:
   ```json
   {
     "name": "titles",
     "version": "1.0.0",
     "private": true,
     "scripts": { "test": "node --test" }
   }
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

2. From `main`, create branch `feat/gh-21-slug-length` and commit one
   change (the request, GitHub issue #21: "slugs should be at most 60
   characters"). Replace `slugify` in `lib/slug.js` with:
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

3. This is an unattended run: nobody will answer questions, so work to
   the end without stopping to ask. Then: **ship this — open a pull
   request.** (No base branch is specified on purpose. `gh` cannot
   reach GitHub in this sandbox; if a `gh` command fails, say so and
   carry on.) End your reply with the ship report and the full drafted
   PR description. Also save that description, exactly as in your
   reply, to `pr-body.md` in the repo root, and leave it untracked.
