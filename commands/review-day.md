---
description: End-of-day review — check task completion, write reflection, archive the day
allowed-tools: Bash(curl:*), Bash(date:*), Bash(cp:*), Bash(mkdir:*), Read, Write, Edit, AskUserQuestion
---

# Review Day

End-of-day review routine. Follow these steps exactly.

## Configuration

- `TODOIST_API_TOKEN` — **required**
- `DAILY_PLANNER_HOME` — default current working directory
- `DAILY_PLANNER_TIMEZONE` — default `UTC`

If `TODOIST_API_TOKEN` is unset, tell the user to set it and stop.

## Step 1: Read today's plan

Read `$DAILY_PLANNER_HOME/today.md`.

## Step 2: Check Todoist completion

```bash
# Active tasks for today
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=today%7Coverdue"

# Completed tasks since today 00:00 UTC
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks/completed?since=$(date -u +%Y-%m-%d)T00:00:00Z"
```

Cross-reference completed list with tasks in `today.md`.

## Step 3: Update today.md checkboxes

- Completed → `- [x]`
- Skipped/cancelled → `- [-]`
- Incomplete → leave `- [ ]`
- Past calendar events → mark `- [x]` (assumed attended unless user says otherwise)

## Step 3b: Close completed tasks in Todoist

For tasks the user marked done in `today.md` that are still **open** in Todoist, close them:

```bash
curl -s -X POST "https://api.todoist.com/api/v1/tasks/<task_id>/close" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"
```

For tasks marked **skipped** (`- [-]`) that are still open, ask the user: close anyway, reschedule, or delete?

```bash
# Reschedule to tomorrow:
curl -s -X POST "https://api.todoist.com/api/v1/tasks/<task_id>" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"due_string": "tomorrow"}'

# Delete:
curl -s -X DELETE "https://api.todoist.com/api/v1/tasks/<task_id>" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN"
```

Match tasks by `(todoist:<id>)` suffix if present, otherwise by content (case-insensitive). If ambiguous, ask.

## Step 4: Ask the user for reflection

Use `AskUserQuestion` to gather:
1. Mood today? (1–5 or a word)
2. Energy level? (1–5 or a word)
3. Any key wins?
4. Any blockers?
5. Anything to carry forward to tomorrow?

## Step 5: Fill the End of Day Review section

Update `today.md`:
- Completed count: X/Y tasks
- Mood, energy from user
- Key wins
- Blockers
- Carry forward (these get picked up by next `/plan-today`)

## Step 6: Archive

Copy completed `today.md` → `$DAILY_PLANNER_HOME/days/{today}.md`. Create the `days/` dir if missing. Then clear `today.md` content (leave a one-line note: "Run /plan-today to start tomorrow").

## Step 7: Show summary

Display:
- Tasks completed vs total
- Carry-forward items
- A short encouraging note

## Important

- Always ask the user before marking uncertain items.
- Never delete user's notes from `today.md` — only fill review fields and update checkboxes.
- Todoist API base: `https://api.todoist.com/api/v1`.
