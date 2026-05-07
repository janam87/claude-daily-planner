---
description: Create a Google Calendar event (with location, attendees, reminders, recurrence)
argument-hint: <natural-language event with time, location, attendees, etc.>
allowed-tools: Bash(date:*), Read, Edit
---

# Add Event

Create a Google Calendar event. User input: `$ARGUMENTS`

## Configuration

- `DAILY_PLANNER_TIMEZONE` — default `UTC`
- `DAILY_PLANNER_CALENDAR_ID` — default `primary`
- `DAILY_PLANNER_HOME` — default current working directory

Requires the **Google Calendar connector** to be enabled in Claude (Settings → Connectors). If unavailable, tell the user and stop.

## Step 1: Parse input

Extract:
- **Summary** (title) — required
- **Start** — date/time (e.g., `tomorrow 3pm`, `2026-05-10 14:00`)
- **End** — explicit (`to 4pm`) or duration (`for 1h`)
- **Location** — `at <place>` or `loc: <address>`
- **Description / notes** — `note: ...`
- **Attendees** — `with email1@x.com, email2@x.com` or `@person@email`
- **Recurrence** — `every weekday`, `weekly`, `monthly` → RFC 5545 RRULE
- **Reminders** — `remind 15m before` → popup; `remind 1h email` → email reminder
- **Conference** — `with meet` or `add zoom` → set conferenceData

Examples:
- `/add-event Team standup tomorrow 9am for 30m every weekday with meet`
- `/add-event Dentist appointment 2026-05-15 11am for 1h at "123 Main St" remind 1h before`
- `/add-event Lunch with Sam thursday 1pm with sam@example.com note: discuss project`

## Step 2: Resolve datetimes

All times in `$DAILY_PLANNER_TIMEZONE`. Convert relative phrases to ISO 8601:
- `tomorrow 9am` → `<tomorrow_date>T09:00:00`
- `for 30m` → end = start + 30 minutes
- All-day events use `date` only (no time component)

## Step 3: Create the event

Use the connector tool:

```
mcp__claude_ai_Google_Calendar__create_event
```

with arguments:

```json
{
  "calendarId": "$DAILY_PLANNER_CALENDAR_ID",
  "summary": "Team standup",
  "description": "...",
  "location": "...",
  "start": {
    "dateTime": "2026-05-08T09:00:00",
    "timeZone": "America/New_York"
  },
  "end": {
    "dateTime": "2026-05-08T09:30:00",
    "timeZone": "America/New_York"
  },
  "attendees": [
    {"email": "sam@example.com"}
  ],
  "recurrence": ["RRULE:FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR"],
  "reminders": {
    "useDefault": false,
    "overrides": [
      {"method": "popup", "minutes": 15}
    ]
  },
  "conferenceData": {
    "createRequest": {
      "requestId": "<random>",
      "conferenceSolutionKey": {"type": "hangoutsMeet"}
    }
  }
}
```

If the connector returns a conferenceData URL, capture it.

## Step 4: Update today.md (if applicable)

If the event is today and `$DAILY_PLANNER_HOME/today.md` exists, add a line to the Schedule section:

```
- [ ] HH:MM–HH:MM — <summary> @ <location>
```

## Step 5: Confirm

Show:
- Event title + time + duration
- Location, attendees (if any)
- Meet/Zoom link (if created)
- Calendar URL: `htmlLink` field from the response
- Whether `today.md` was updated

## Important

- All times in `$DAILY_PLANNER_TIMEZONE`. The calendar stores in UTC under the hood.
- Recurrence uses RFC 5545 RRULE format.
- Attendees get email invites unless `sendUpdates: "none"` is passed — ask before inviting external emails.
- The connector requires OAuth — re-auth if 401 is returned.
