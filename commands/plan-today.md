---
description: Morning routine — generate today's daily plan from Google Calendar, Todoist, and yesterday's carry-forwards
allowed-tools: Bash(curl:*), Bash(date:*), Read, Write, Edit
---

# Plan Today

Generate today's daily plan. Follow these steps exactly.

## Configuration

Read these environment variables (with sensible defaults):

- `TODOIST_API_TOKEN` — **required** for Todoist (https://app.todoist.com/app/settings/integrations/developer)
- `DAILY_PLANNER_TIMEZONE` — default `UTC`
- `DAILY_PLANNER_HOME` — default current working directory (where `today.md`, `week.md`, `days/` live)
- `DAILY_PLANNER_CALENDAR_ID` — default `primary`

If `TODOIST_API_TOKEN` is unset, tell the user to set it (see plugin INTEGRATIONS.md) and stop.

## Step 1: Get today's date

Use the current date in `$DAILY_PLANNER_TIMEZONE`. If the var is unset, use `UTC`.

## Step 2: Check yesterday's archive

Read `$DAILY_PLANNER_HOME/days/{yesterday}.md` if it exists. Extract carry-forward items and any unchecked `- [ ]` items.

## Step 3: Fetch Google Calendar events

Use the `mcp__claude_ai_Google_Calendar__list_events` MCP tool (requires the Google Calendar connector enabled in Claude). Pass:

- `timeMin`: today 00:00:00 in `$DAILY_PLANNER_TIMEZONE`
- `timeMax`: today 23:59:59 in `$DAILY_PLANNER_TIMEZONE`
- `timeZone`: `$DAILY_PLANNER_TIMEZONE`
- `calendarId`: `$DAILY_PLANNER_CALENDAR_ID` (or `primary`)

If the connector is not available, skip this step and warn the user.

## Step 4: Fetch Todoist tasks

```bash
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=today%7Coverdue"
```

The Todoist npm MCP package uses the deprecated v2 API — always call `/api/v1/` directly via curl.

## Step 5: Generate today.md

Read the template at `${CLAUDE_PLUGIN_ROOT}/templates/daily.md`. Write the populated file to `$DAILY_PLANNER_HOME/today.md` with:

1. **Schedule** — merged calendar events + free blocks labeled "Focus time"; default wake 07:15, sleep 23:00 if no sleep events.
2. **Tasks**:
   - **Priority**: P1 + P2 (Todoist priority `4` and `3`)
   - **Other**: P3, P4, unset (priority `2`, `1`)
   - Mark carry-forward items with `(carried forward)`
3. **Notes** — leave empty.
4. **End of Day Review** — leave empty.

## Step 6: Show the plan

Display the generated plan to the user.

## Important

- Do NOT overwrite `today.md` if it already has user notes/checkmarks. Ask first.
- Always fetch fresh data from Calendar and Todoist.
- Todoist API base: `https://api.todoist.com/api/v1` (not v2 — deprecated).
- Priority mapping: Todoist `4` = P1 (highest), `3` = P2, `2` = P3, `1` = P4/none.
