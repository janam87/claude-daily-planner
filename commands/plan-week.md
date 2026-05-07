---
description: Weekly planning — generate week overview from Google Calendar and Todoist
allowed-tools: Bash(curl:*), Bash(date:*), Read, Write, Edit
---

# Plan Week

Generate a weekly plan. Follow these steps exactly.

## Configuration

Read these environment variables:

- `TODOIST_API_TOKEN` — **required**
- `DAILY_PLANNER_TIMEZONE` — default `UTC`
- `DAILY_PLANNER_HOME` — default current working directory
- `DAILY_PLANNER_CALENDAR_ID` — default `primary`

If `TODOIST_API_TOKEN` is unset, tell the user to set it (see plugin INTEGRATIONS.md) and stop.

## Step 1: Determine the week

Calculate Monday → Sunday of the current week in `$DAILY_PLANNER_TIMEZONE`.

## Step 2: Review past days

Read any existing `$DAILY_PLANNER_HOME/days/*.md` for this week to understand what's already done or carried forward.

## Step 3: Fetch Google Calendar events for the week

Use `mcp__claude_ai_Google_Calendar__list_events` with:

- `timeMin`: Monday 00:00:00
- `timeMax`: Sunday 23:59:59
- `timeZone`: `$DAILY_PLANNER_TIMEZONE`
- `calendarId`: `$DAILY_PLANNER_CALENDAR_ID`

## Step 4: Fetch Todoist tasks

```bash
# Tasks due in next 7 days
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=7%20days"

# Overdue
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=overdue"
```

## Step 5: Generate week.md

Read template at `${CLAUDE_PLUGIN_ROOT}/templates/weekly.md`. Write `$DAILY_PLANNER_HOME/week.md` with:

1. **Weekly Goals** — leave for user, or suggest based on high-priority tasks.
2. **Day Overview** table — fill each day with key events + tasks.
3. **Tasks by priority** — Must Do (P1/P2), Should Do (P3/P4), Overdue.
4. **Weekly Review** — leave empty.

## Step 6: Show the plan

Display the weekly overview.

## Important

- If `week.md` already exists with user content, ask before overwriting.
- Highlight scheduling conflicts.
- Flag overloaded days.
- Priority mapping: `4`=P1, `3`=P2, `2`=P3, `1`=P4/none.
