# Daily Planner — Plugin Conventions

This file is loaded automatically when the `daily-planner` plugin is active. It defines conventions all commands share.

## Environment Variables

| Variable | Required | Default | Purpose |
|----------|----------|---------|---------|
| `TODOIST_API_TOKEN` | Yes | — | Bearer token for Todoist REST API |
| `DAILY_PLANNER_HOME` | No | `$PWD` | Where `today.md`, `week.md`, `days/` live |
| `DAILY_PLANNER_TIMEZONE` | No | `UTC` | IANA timezone for date math (e.g. `America/New_York`, `Europe/London`) |
| `DAILY_PLANNER_CALENDAR_ID` | No | `primary` | Google Calendar ID to read/write |

If `TODOIST_API_TOKEN` is missing, every Todoist-touching command stops and instructs the user to set it (see `INTEGRATIONS.md`).

## File Layout (in `$DAILY_PLANNER_HOME`)

- `today.md` — active daily plan. Regenerated each morning via `/plan-today`.
- `week.md` — weekly overview. Regenerated via `/plan-week`.
- `days/YYYY-MM-DD.md` — archived daily logs with full status.
- Templates ship inside the plugin at `${CLAUDE_PLUGIN_ROOT}/templates/`.

## Sync Direction

The plugin is **bidirectional**:

| Direction | Commands |
|-----------|----------|
| Pull (services → markdown) | `/plan-today`, `/plan-week` |
| Push (markdown → services) | `/sync-today`, `/quick-add`, `/add-event`, `/create-project` |
| Two-way (mark done both sides) | `/review-day` |
| Read-only review | `/review-yesterday`, `/review-week` |

Lines tagged with `(todoist:<id>)` or `(gcal:<eventId>)` are considered already synced — never re-created.

## Todoist API

- Base: `https://api.todoist.com/api/v1` (NOT v2 — deprecated).
- Auth: `Authorization: Bearer $TODOIST_API_TOKEN`.
- The npm `@doist/todoist-mcp` package targets v2 and does **not** work — always call v1 directly via curl.

### Read endpoints

| Action | Method | Path |
|--------|--------|------|
| List tasks | GET | `/tasks?filter=today\|overdue` |
| List projects | GET | `/projects` |
| List sections | GET | `/sections?project_id=<id>` |
| List labels | GET | `/labels` |
| Search tasks | GET | `/tasks?filter=search:<query>` |
| Completed today | GET | `/tasks/completed?since=<ISO>` |

### Write endpoints

| Action | Method | Path |
|--------|--------|------|
| Create task | POST | `/tasks` |
| Update task | POST | `/tasks/{id}` |
| Close task | POST | `/tasks/{id}/close` |
| Reopen task | POST | `/tasks/{id}/reopen` |
| Delete task | DELETE | `/tasks/{id}` |
| Create project | POST | `/projects` |
| Update project | POST | `/projects/{id}` |
| Delete project | DELETE | `/projects/{id}` |
| Create section | POST | `/sections` |
| Create label | POST | `/labels` |
| Add comment / note | POST | `/comments` |

### Task body fields (POST `/tasks`)

```json
{
  "content": "string",
  "description": "string (notes)",
  "project_id": "string",
  "section_id": "string",
  "parent_id": "string (for subtasks)",
  "priority": 1,
  "labels": ["string"],
  "due_string": "tomorrow 4pm",
  "due_date": "2026-05-08",
  "due_datetime": "2026-05-08T16:00:00Z",
  "due_lang": "en",
  "duration": 30,
  "duration_unit": "minute",
  "assignee_id": "string"
}
```

Priority mapping: Todoist `4` = P1 (highest), `3` = P2, `2` = P3, `1` = P4/none.

### Reminders & locations (Sync API, **Pro only**)

Reminders are not part of REST v1. Use `POST /api/v1/sync` with a `reminder_add` command:

```json
{
  "commands": [{
    "type": "reminder_add",
    "temp_id": "<uuid>",
    "uuid": "<uuid>",
    "args": {
      "item_id": "<task_id>",
      "type": "relative|absolute|location",
      "minute_offset": 30,
      "due": {"date": "2026-05-08T16:00:00"},
      "loc_lat": "40.7128",
      "loc_long": "-74.0060",
      "loc_trigger": "on_enter|on_leave",
      "radius": 100
    }
  }]
}
```

If the user is on free Todoist, reminders silently no-op — warn them.

## Google Calendar

Uses the Claude built-in connector — no separate API key.

### Read

- `mcp__claude_ai_Google_Calendar__list_events` — list within a time range
- `mcp__claude_ai_Google_Calendar__get_event` — single event
- `mcp__claude_ai_Google_Calendar__list_calendars` — discover calendar IDs
- `mcp__claude_ai_Google_Calendar__suggest_time` — find free slots

### Write

- `mcp__claude_ai_Google_Calendar__create_event` — new event
- `mcp__claude_ai_Google_Calendar__update_event` — modify event
- `mcp__claude_ai_Google_Calendar__delete_event` — remove event
- `mcp__claude_ai_Google_Calendar__respond_to_event` — RSVP to invite

Pass `timeZone: $DAILY_PLANNER_TIMEZONE` and `calendarId: $DAILY_PLANNER_CALENDAR_ID`. If the connector is not enabled, calendar steps skip with a warning.

## Conventions

- All times in `$DAILY_PLANNER_TIMEZONE`.
- Daily files: `YYYY-MM-DD.md`.
- Checkboxes: `- [ ]` pending, `- [x]` done, `- [-]` skipped/cancelled.
- Carry-forward items from prior day get suffix `(carried forward)`.
- Synced items get suffix `(todoist:<id>)` or `(gcal:<eventId>)` — used for idempotent re-sync.

## Commands

| Command | Direction | Purpose |
|---------|-----------|---------|
| `/plan-today` | Pull | Morning: generate today's plan from Calendar + Todoist |
| `/plan-week` | Pull | Weekly: generate week overview |
| `/sync-today` | Push | Push hand-edited `today.md` back to Calendar + Todoist |
| `/quick-add <task>` | Push | Add Todoist task with full fields (project, due, priority, labels, subtasks, reminders) |
| `/add-event <event>` | Push | Create Google Calendar event |
| `/create-project <name>` | Push | Create Todoist project |
| `/review-day` | Two-way | Night: mark done in Todoist + local, ask reflection, archive |
| `/review-yesterday` | Read | Summary of yesterday's archive |
| `/review-week` | Read | Aggregate stats across the week |
