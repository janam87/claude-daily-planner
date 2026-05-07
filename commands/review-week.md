---
description: Review all days this week — show completion summary, patterns, and productivity
allowed-tools: Bash(date:*), Bash(ls:*), Read, AskUserQuestion
---

# Review Week

Review the entire week's progress. Follow these steps.

## Configuration

- `DAILY_PLANNER_HOME` — default current working directory
- `DAILY_PLANNER_TIMEZONE` — default `UTC`

## Step 1: Determine the week range

Calculate Monday → Sunday (or today if mid-week) in `$DAILY_PLANNER_TIMEZONE`.

## Step 2: Read all daily archives

Read every file in `$DAILY_PLANNER_HOME/days/` whose date falls in this week.

## Step 3: Aggregate data

Across all days:
- Total tasks completed vs total planned
- Tasks by priority completed
- Carry-forward frequency (items that kept getting pushed)
- Calendar events attended
- Mood + energy trends (if recorded)

## Step 4: Show weekly summary

```
## Week Review — {Mon Date} to {Sun Date}

### Completion Rate
- Tasks: X/Y completed (Z%)
- Calendar events: X attended

### Daily Breakdown
| Day | Planned | Done | Mood | Energy |
|-----|---------|------|------|--------|
| Mon | X | Y | ... | ... |
| Tue | X | Y | ... | ... |

### Recurring Carry-Forwards
(Items incomplete on 2+ days — need attention)
- item 1 (incomplete X days)

### Mood & Energy Trend
Mon: X/5 → Tue: X/5 → ...

### Highlights
- Best day: {day} (most tasks done)
- Toughest day: {day} (most incomplete)
```

## Step 5: Offer actions

- Want to plan next week? (suggest `/plan-week`)
- Recurring blockers to address?
- Tasks to delete/reschedule in Todoist?

## Important

- Read-only on archive files — never modify `days/*.md`.
- If no archives exist for the week, tell the user and suggest `/plan-today`.
