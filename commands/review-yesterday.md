---
description: Review yesterday's archived daily plan — show what was done, pending, incomplete
allowed-tools: Bash(date:*), Read, AskUserQuestion
---

# Review Yesterday

Review the previous day's archived plan. Follow these steps.

## Configuration

- `DAILY_PLANNER_HOME` — default current working directory
- `DAILY_PLANNER_TIMEZONE` — default `UTC`

## Step 1: Find yesterday's archive

Read `$DAILY_PLANNER_HOME/days/{yesterday}.md`. If missing, tell the user there's no archive for yesterday and offer to run `/review-day` if `today.md` was actually yesterday's plan.

## Step 2: Parse the file

Extract:
- **Completed** (`- [x]`)
- **Incomplete** (`- [ ]`)
- **Skipped** (`- [-]`)
- **Carry-forward items** (from End of Day Review)
- **Notes** the user wrote
- **Mood and energy** from review section

## Step 3: Show summary

```
## Yesterday — {Day, Date}

### Done (X items)
- [x] item 1

### Not Done (X items)
- [ ] item 1

### Skipped (X items)
- [-] item 1

### Carry Forward
- item 1

### Mood: X | Energy: X
### Notes: (any notes)
```

## Step 4: Offer actions

Ask the user (use `AskUserQuestion`):
- Carry forward incomplete items to today? (updates `today.md` if it exists)
- Reschedule any items in Todoist?
- Mark any incomplete items as done now?

## Important

- This is **read-only** on the archive — never modify files in `days/`.
- Only modify `today.md` if the user explicitly asks to carry items forward.
