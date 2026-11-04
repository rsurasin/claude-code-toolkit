---
max_turns: 30
allowed_tools: [Read, Glob, Grep, Skill, Bash, Write, Edit, Agent]
---

Set up a scratch project with an active task, then settle a product
question and keep working:

1. Create a temporary directory with `git init`, a `.gitignore`
   containing `.claude/dev/`, an initial commit with a small `logs.py`,
   and a `feat/gh-31-workout-logs` branch checked out.
2. Create the task folder
   `.claude/dev/features/2026-10-04-gh-31-workout-logs/` with `plan.md`
   (including a `## Key Decisions` section), `context.md` (with a few
   existing Session Log entries), and `tasks.md`.
3. Then, as the user: **The app can send a workout log for an exercise
   we've since retired from the plan. Two options: reject those logs
   after a 7-day grace period, which keeps reports clean but throws away
   workouts people did offline; or accept them and keep the retired
   exercise name on the log, which never loses data but means reports can
   show exercises no longer in the plan. Let's accept them: losing a
   user's workout is worse than a messy report. Now add an
   `accept_log(log)` function to `logs.py` that stores the log as is.**
