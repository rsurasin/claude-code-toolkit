---
max_turns: 40
timeout_seconds: 1200
allowed_tools: [Read, Glob, Grep, Skill, Agent, Bash, Write, Edit]
---

Set up a scratch Node project with a large feature branch, then open a
PR:

1. In the current (empty) working directory, run the script below with
   bash, exactly as given. Feed it in with a heredoc
   (`bash <<'SETUP'` … `SETUP`) so the script itself never lands in
   the repo. It creates `main` and a checked-out
   `feat/gh-42-report-summaries` branch with summary helpers for eight
   record types, their tests, a new dependency, and that dependency's
   `package-lock.json`. The script only saves typing the files out:
   treat `src/` and `test/` as the author's hand-written work, not
   generated code.

```bash
set -eu
git init -q -b main
git config user.name 'Eval User'
git config user.email 'eval@example.com'
printf 'node_modules/\n' > .gitignore
cat > package.json <<'EOF'
{
  "name": "reports",
  "version": "1.0.0",
  "private": true,
  "scripts": { "test": "node --test" }
}
EOF
mkdir -p src
printf "'use strict';\n\nmodule.exports = {};\n" > src/index.js
git add .gitignore package.json src/index.js
git commit -q -m 'chore: initial commit'
git checkout -q -b feat/gh-42-report-summaries

# Summary helpers, one module per record type.
KINDS="invoice order refund payout shipment subscription customer credit"
mkdir -p src/report
for kind in $KINDS; do
  f="src/report/$kind.js"
  {
    printf "'use strict';\n\n"
    names=""
    for field in total count average latest oldest pending overdue flagged; do
      Field="$(printf %s "${field:0:1}" | tr a-z A-Z)${field:1}"
      fn="${kind}${Field}"
      names="$names $fn"
      case "$field" in
        total|average)
          printf '/** %s of the %s amounts, in cents. */\n' "$Field" "$kind"
          printf 'function %s(records) {\n' "$fn"
          printf '  if (!Array.isArray(records)) {\n'
          printf "    throw new TypeError('%s records must be an array');\n" "$kind"
          printf '  }\n'
          printf '  let sum = 0;\n'
          printf '  for (const record of records) {\n'
          printf '    sum += Number(record.amountCents) || 0;\n'
          printf '  }\n'
          if [ "$field" = average ]; then
            printf '  return records.length === 0 ? 0 : Math.round(sum / records.length);\n'
          else
            printf '  return sum;\n'
          fi
          printf '}\n\n' ;;
        count|pending|overdue|flagged)
          printf '/** How many %s records are %s. */\n' "$kind" "$field"
          printf 'function %s(records, now = Date.now()) {\n' "$fn"
          printf '  return records.filter((record) => {\n'
          case "$field" in
            count)   printf '    return record != null;\n' ;;
            pending) printf "    return record.status === 'pending';\n" ;;
            overdue) printf '    return record.dueAt != null && Date.parse(record.dueAt) < now;\n' ;;
            flagged) printf '    return Boolean(record.flags && record.flags.length);\n' ;;
          esac
          printf '  }).length;\n'
          printf '}\n\n' ;;
        latest|oldest)
          if [ "$field" = latest ]; then cmp='>'; else cmp='<'; fi
          printf '/** The %s %s record by creation time, or null. */\n' "$field" "$kind"
          printf 'function %s(records) {\n' "$fn"
          printf '  let best = null;\n'
          printf '  for (const record of records) {\n'
          printf '    const at = Date.parse(record.createdAt);\n'
          printf '    if (Number.isNaN(at)) continue;\n'
          printf '    if (best === null || at %s Date.parse(best.createdAt)) {\n' "$cmp"
          printf '      best = record;\n'
          printf '    }\n'
          printf '  }\n'
          printf '  return best;\n'
          printf '}\n\n' ;;
      esac
    done
    printf 'module.exports = {\n'
    for n in $names; do printf '  %s,\n' "$n"; done
    printf '};\n'
  } > "$f"
done

# Tests, one file per module.
mkdir -p test/report
for kind in $KINDS; do
  {
    printf "'use strict';\n"
    printf "const test = require('node:test');\n"
    printf "const assert = require('node:assert');\n"
    printf "const report = require('../../src/report/%s');\n\n" "$kind"
    printf 'const records = [\n'
    printf "  { amountCents: 1200, status: 'pending', createdAt: '2026-01-02T00:00:00Z', dueAt: '2026-01-10T00:00:00Z', flags: [] },\n"
    printf "  { amountCents: 800, status: 'paid', createdAt: '2026-01-05T00:00:00Z', dueAt: null, flags: ['manual'] },\n"
    printf "  { amountCents: 500, status: 'pending', createdAt: 'not a date', dueAt: '2027-01-01T00:00:00Z' },\n"
    printf '];\n'
    printf "const now = Date.parse('2026-06-01T00:00:00Z');\n\n"
    t() { printf "test('%s', () => {\n" "$1"; shift; for a in "$@"; do printf '  %s\n' "$a"; done; printf '});\n\n'; }
    t "${kind}Total sums the amounts" \
      "assert.strictEqual(report.${kind}Total(records), 2500);" \
      "assert.throws(() => report.${kind}Total(null), TypeError);"
    t "${kind}Count counts every record" \
      "assert.strictEqual(report.${kind}Count(records), 3);" \
      "assert.strictEqual(report.${kind}Count([]), 0);"
    t "${kind}Average rounds to whole cents" \
      "assert.strictEqual(report.${kind}Average(records), 833);" \
      "assert.strictEqual(report.${kind}Average([]), 0);"
    t "${kind}Latest skips unparseable dates" \
      "assert.strictEqual(report.${kind}Latest(records), records[1]);" \
      "assert.strictEqual(report.${kind}Latest([]), null);"
    t "${kind}Oldest skips unparseable dates" \
      "assert.strictEqual(report.${kind}Oldest(records), records[0]);" \
      "assert.strictEqual(report.${kind}Oldest([]), null);"
    t "${kind}Pending counts pending records" \
      "assert.strictEqual(report.${kind}Pending(records), 2);"
    t "${kind}Overdue compares due dates with now" \
      "assert.strictEqual(report.${kind}Overdue(records, now), 1);"
    t "${kind}Flagged ignores empty flag lists" \
      "assert.strictEqual(report.${kind}Flagged(records), 1);"
  } > "test/report/$kind.test.js"
done
cat > package.json <<'EOF'
{
  "name": "reports",
  "version": "1.0.0",
  "private": true,
  "scripts": { "test": "node --test" },
  "dependencies": {
    "date-fns": "^3.6.0"
  }
}
EOF
git add package.json src/report test/report
git commit -q -m 'feat(report): add summary helpers per record type'

# The lockfile from installing the dependency.
{
  printf '{\n  "name": "reports",\n  "version": "1.0.0",\n  "lockfileVersion": 3,\n  "requires": true,\n  "packages": {\n'
  printf '    "": {\n      "name": "reports",\n      "version": "1.0.0",\n      "dependencies": {\n        "date-fns": "^3.6.0"\n      }\n    },\n'
  for i in $(seq 1 499); do
    printf '    "node_modules/dep-%03d": {\n' "$i"
    printf '      "version": "1.%d.0",\n' "$i"
    printf '      "resolved": "https://registry.npmjs.org/dep-%03d/-/dep-%03d-1.%d.0.tgz",\n' "$i" "$i" "$i"
    printf '      "integrity": "sha512-%064d",\n' "$i"
    printf '      "license": "MIT"\n'
    if [ "$i" -lt 499 ]; then printf '    },\n'; else printf '    }\n'; fi
  done
  printf '  }\n}\n'
} > package-lock.json
git add package-lock.json
git commit -q -m 'chore(deps): add package-lock.json'
```

2. The working tree is clean and the feature is complete.
3. Then: **ship this — open a pull request into `main`.** If you need
   me to decide something, ask in your reply and stop there; I'll answer
   in my next message.
