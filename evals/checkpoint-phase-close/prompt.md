---
max_turns: 80
timeout_seconds: 1800
allowed_tools: [Read, Glob, Grep, Skill, Agent, Bash, Write, Edit]
---

Set up a scratch Node project partway through a phased feature, then
commit the next task:

1. In the current (empty) working directory, run the script below with
   bash, exactly as given. Feed it in with a heredoc
   (`bash <<'SETUP'` … `SETUP`) so the script itself never lands in
   the repo. It creates `main` and a checked-out
   `feat/gh-51-booking-window` branch (GitHub issue #51, "booking
   window helpers") with three commits, and leaves the next task's
   change in the working tree, uncommitted. There is no remote.

```bash
set -eu
git init -q -b main
git config user.name 'Eval User'
git config user.email 'eval@example.com'
printf 'node_modules/\n' > .gitignore
cat > package.json <<'EOF'
{
  "name": "bookings",
  "version": "1.0.0",
  "private": true,
  "scripts": { "test": "node --test" }
}
EOF
git add .gitignore package.json
git commit -q -m 'chore: initial commit'
git checkout -q -b feat/gh-51-booking-window
mkdir -p lib test

# Phase "Date parsing": its one task.
cat > lib/dates.js <<'EOF'
'use strict';

/** Parse an ISO 8601 date (YYYY-MM-DD) into a UTC Date, or throw. */
function parseDate(text) {
  if (!/^\d{4}-\d{2}-\d{2}$/.test(text)) {
    throw new RangeError(`not an ISO date: ${text}`);
  }
  const date = new Date(`${text}T00:00:00Z`);
  if (Number.isNaN(date.getTime())) {
    throw new RangeError(`not a real date: ${text}`);
  }
  return date;
}

module.exports = { parseDate };
EOF
cat > test/dates.test.js <<'EOF'
'use strict';
const test = require('node:test');
const assert = require('node:assert');
const { parseDate } = require('../lib/dates');

test('parses an ISO date as UTC midnight', () => {
  assert.strictEqual(parseDate('2026-03-01').toISOString(), '2026-03-01T00:00:00.000Z');
});

test('rejects text that is not an ISO date', () => {
  assert.throws(() => parseDate('03/01/2026'), RangeError);
});
EOF
git add lib/dates.js test/dates.test.js
git commit -q -m 'feat(dates): parse ISO dates'

# Phase "Booking window", first task.
cat > lib/window.js <<'EOF'
'use strict';
const { parseDate } = require('./dates');

/**
 * Whether `day` falls inside the booking window from `start` to `end`,
 * inclusive of both ends. All three are ISO dates (YYYY-MM-DD).
 */
function isWithinWindow(day, start, end) {
  const at = parseDate(day).getTime();
  return at > parseDate(start).getTime() && at < parseDate(end).getTime();
}

module.exports = { isWithinWindow };
EOF
cat > test/window.test.js <<'EOF'
'use strict';
const test = require('node:test');
const assert = require('node:assert');
const window = require('../lib/window');

test('a day in the middle of the window is inside it', () => {
  assert.strictEqual(window.isWithinWindow('2026-03-05', '2026-03-01', '2026-03-10'), true);
});

test('a day after the window is outside it', () => {
  assert.strictEqual(window.isWithinWindow('2026-03-11', '2026-03-01', '2026-03-10'), false);
});
EOF
git add lib/window.js test/window.test.js
git commit -q -m 'feat(window): check a day against a booking window'

# Phase "Booking window", next task: left uncommitted.
cat > lib/window.js <<'EOF'
'use strict';
const { parseDate } = require('./dates');

const DAY_MS = 24 * 60 * 60 * 1000;

/**
 * Whether `day` falls inside the booking window from `start` to `end`,
 * inclusive of both ends. All three are ISO dates (YYYY-MM-DD).
 */
function isWithinWindow(day, start, end) {
  const at = parseDate(day).getTime();
  return at > parseDate(start).getTime() && at < parseDate(end).getTime();
}

/** Whole days from `today` until `day` (negative once it has passed). */
function daysUntil(day, today) {
  return Math.round((parseDate(day).getTime() - parseDate(today).getTime()) / DAY_MS);
}

module.exports = { isWithinWindow, daysUntil };
EOF
cat >> test/window.test.js <<'EOF'

test('counts the days until a booking', () => {
  assert.strictEqual(window.daysUntil('2026-03-10', '2026-03-01'), 9);
  assert.strictEqual(window.daysUntil('2026-02-27', '2026-03-01'), -2);
});
EOF
```

2. The plan for issue #51 was estimated at about 900 changed lines, so
   it was split into phases. The task files aren't available here, so
   this is the phase plan:

   - **Date parsing**: one task, "parse ISO dates". Committed, and its
     checkpoint is done.
   - **Booking window**: started at the commit
     `feat(dates): parse ISO dates` (the phase's start commit), with
     two tasks: "check a day against a booking window" (committed) and
     "count the days until a booking" (the change now in the working
     tree). This is the phase's last task.
   - **Booking API**: not started, so the feature is not finished.

3. Then: **commit this.**
