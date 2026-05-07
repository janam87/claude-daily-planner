---
description: Push today.md schedule blocks to Google Calendar and tasks to Todoist (reverse direction of /plan-today)
allowed-tools: Bash(curl:*), Bash(date:*), Read, Edit, AskUserQuestion
---

# Sync Today

Push the user's hand-edited `today.md` back to Google Calendar and Todoist. This is the **reverse** of `/plan-today` — for when the user has tweaked `today.md` directly and wants the changes reflected upstream.

## Configuration

- `TODOIST_API_TOKEN` — **required**
- `DAILY_PLANNER_HOME` — default current working directory
- `DAILY_PLANNER_TIMEZONE` — default `UTC`
- `DAILY_PLANNER_CALENDAR_ID` — default `primary`

## Step 1: Read today.md

Read `$DAILY_PLANNER_HOME/today.md`. Parse:

- **Schedule blocks**: lines like `- [ ] HH:MM–HH:MM — <summary>` (with optional `@ <location>`)
- **Priority tasks**: under `### Priority` header → P1/P2
- **Other tasks**: under `### Other` header → P3/P4
- Tasks already linked to Todoist will have a `(todoist:<id>)` suffix — skip those (already synced)

## Step 2: Diff against current state

Fetch current state of both services:

```bash
# Today's calendar events
mcp__claude_ai_Google_Calendar__list_events  # for today

# Today's Todoist tasks
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=today%7Coverdue"
```

Build three lists:
- **New in today.md, not upstream** → create
- **Upstream, not in today.md** → ask user (delete? leave?)
- **In both, but content differs** → ask user (which side wins?)

## Step 3: Confirm with user

Use `AskUserQuestion` to show the diff and confirm before writing. Never auto-delete — only auto-create new items.

## Step 4: Push schedule blocks → Calendar

For each new schedule block, call `mcp__claude_ai_Google_Calendar__create_event` (see `/add-event` for full payload structure).

After creation, append `(gcal:<eventId>)` to the schedule line in `today.md` so re-syncs are idempotent.

## Step 5: Push tasks → Todoist

For each new task line:

```bash
curl -s -X POST "https://api.todoist.com/api/v1/tasks" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "...",
    "priority": N,
    "due_string": "today",
    "labels": ["..."]
  }'
```

After creation, append `(todoist:<id>)` to the task line in `today.md`.

## Step 6: Update today.md with IDs

Write the modified `today.md` back so future syncs are idempotent (lines with IDs are skipped).

## Step 7: Show summary

```
Synced X schedule blocks → Calendar (Y created, Z skipped)
Synced X tasks → Todoist (Y created, Z skipped)
```

If anything was ambiguous, list it and ask follow-up.

## Important

- **Idempotent**: lines tagged with `(gcal:...)` or `(todoist:...)` are never re-created.
- Never auto-delete upstream items. Always confirm.
- For two-way sync conflicts (item exists in both with different content), default to local-wins but ask the user.
- Bulk creates can hit rate limits; on `429` back off 5s and retry.
