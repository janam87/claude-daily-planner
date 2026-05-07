---
description: Add a Todoist task with full fields (project, due, priority, labels, description, subtasks, reminders, location) and update today's plan
argument-hint: <natural-language task with optional time/priority/project/labels/notes>
allowed-tools: Bash(curl:*), Bash(date:*), Bash(uuidgen:*), Read, Write, Edit
---

# Quick Add

Create a Todoist task with all supported fields, optionally as a subtask, optionally with a reminder, then update `today.md`. User input: `$ARGUMENTS`

## Configuration

- `TODOIST_API_TOKEN` — **required**
- `DAILY_PLANNER_HOME` — default current working directory

If `TODOIST_API_TOKEN` is unset, tell the user to set it and stop.

## Step 1: Parse the input

Extract from `$ARGUMENTS`:

| Field | How user expresses it | Todoist field |
|-------|----------------------|----------------|
| Content | the main task text | `content` |
| Notes / description | `note: ...` or `-- ...` | `description` |
| Due | "tomorrow", "at 6pm", "every Monday", "next Friday 9am" | `due_string` |
| Priority | `P1` `P2` `P3` `P4` | `priority` (4=P1, 3=P2, 2=P3, 1=P4) |
| Project | `#ProjectName` | `project_id` (resolve via list) |
| Section | `/SectionName` | `section_id` |
| Labels / tags | `@label1` `@label2` | `labels` (array of strings) |
| Parent (subtask) | `under "<task>"` or `>>` after task ID | `parent_id` |
| Duration | `for 30min`, `for 2h` | `duration` + `duration_unit` |
| Reminder | `remind me 30m before`, `remind at 9am` | sync API (Pro only) |
| Location | `at <place>` (only if reminder is location-based) | sync API loc_lat/loc_long |

Examples:
- `/quick-add Call dentist tomorrow 4pm P2 #Personal @health remind me 30m before`
- `/quick-add Fix login bug P1 #Work /Backend @bug for 2h note: see issue #123`
- `/quick-add Buy milk under "Groceries" @errand`

## Step 2: Resolve project / section / parent IDs

If user gave names (not IDs):

```bash
# Projects
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/projects"

# Sections (filtered to project)
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/sections?project_id=<id>"

# Tasks (to find parent task by content match)
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=search:<query>"
```

Match case-insensitively. If no match, ask the user before creating.

## Step 3: Create the task

```bash
curl -s -X POST "https://api.todoist.com/api/v1/tasks" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "...",
    "description": "...",
    "project_id": "...",
    "section_id": "...",
    "parent_id": "...",
    "priority": 3,
    "labels": ["health"],
    "due_string": "tomorrow 4pm",
    "duration": 30,
    "duration_unit": "minute"
  }'
```

Capture the returned task `id` for the next step.

## Step 4: (Optional) Add a reminder

Reminders use the Sync API. **Requires Todoist Pro.** If the user is not on Pro, the call will succeed silently but no reminder fires — warn them once if they ask for reminders and the project look paid-only.

**Time-based reminder:**

```bash
UUID=$(uuidgen)
TEMP=$(uuidgen)
curl -s -X POST "https://api.todoist.com/api/v1/sync" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"commands\": [{
      \"type\": \"reminder_add\",
      \"temp_id\": \"$TEMP\",
      \"uuid\": \"$UUID\",
      \"args\": {
        \"item_id\": \"<task_id>\",
        \"type\": \"relative\",
        \"minute_offset\": 30
      }
    }]
  }"
```

For absolute time use `"type": "absolute"` and `"due": {"date": "2026-05-08T16:00:00"}`.

**Location-based reminder** (Pro):

```json
{
  "type": "reminder_add",
  "args": {
    "item_id": "<task_id>",
    "type": "location",
    "name": "Home",
    "loc_lat": "40.7128",
    "loc_long": "-74.0060",
    "loc_trigger": "on_enter",
    "radius": 100
  }
}
```

Ask the user for coordinates if they only gave a place name (or skip the reminder and warn).

## Step 5: Update today.md

If task is due today and `$DAILY_PLANNER_HOME/today.md` exists:
- Add to **Priority** section (P1/P2) or **Other** section (P3/P4)
- If specific time, also add to **Schedule** section as a time block
- Use checkbox `- [ ]`

## Step 6: Confirm

Show the user:
- Task ID + content
- All fields that were set (project, due, labels, etc.)
- Whether a reminder was attached (or warning if Pro required)
- Whether `today.md` was updated

## Important

- Todoist API base: `https://api.todoist.com/api/v1`
- Priority mapping: P1 → `4`, P2 → `3`, P3 → `2`, P4 → `1`
- Labels are passed as **names**, not IDs (Todoist v1 accepts strings)
- For subtasks, `parent_id` is required and the child inherits the parent's project automatically
- If `today.md` doesn't exist, just create the Todoist task and suggest `/plan-today`
