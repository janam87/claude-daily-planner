# Daily Planner

[![CI](https://github.com/janam87/claude-daily-planner/actions/workflows/ci.yml/badge.svg)](https://github.com/janam87/claude-daily-planner/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-7c3aed.svg)](https://docs.claude.com/en/docs/claude-code/plugins)

A Claude Code plugin that turns Claude into your daily journal + scheduler. **Bidirectional** sync with Google Calendar and Todoist — pull your day in, edit in markdown, push changes back. Asks you for an end-of-day reflection. Archives every day in plain markdown so you own your data forever.

## Demo

```
$ /plan-today

✓ Read yesterday's archive — 2 carry-forward items
✓ Fetched 4 calendar events for today
✓ Fetched 11 Todoist tasks (3 P1, 5 P2, 3 P3)
✓ Wrote ~/journal/today.md

# Monday, May 7, 2026

## Schedule
- [ ] 09:00–09:30 — Team standup @ Meet
- [ ] 10:00–12:00 — Focus time
- [ ] 14:00–15:00 — 1:1 with Sam
- [ ] 16:00–17:00 — Code review

## Tasks
### Priority
- [ ] Ship invoice rewrite (todoist:8123) (carried forward)
- [ ] Reply to legal team (todoist:8124)

### Other
- [ ] Update onboarding doc (todoist:8125)
...
```

Then through the day:

```
$ /quick-add Call dentist tomorrow 4pm P2 #Personal @health remind me 30m before
✓ Created task 8201 in #Personal, due tomorrow 16:00, P2, label @health
✓ Reminder set: 30m before
✓ today.md not updated (task is for tomorrow)
```

End of day:

```
$ /review-day
✓ Marked 9/13 tasks complete in Todoist
✓ Archived to ~/journal/days/2026-05-07.md
✓ Cleared today.md

📊 9/13 done · mood 4/5 · energy 3/5
🎯 carry forward: "ship invoice rewrite"
```

```
~/journal/
├── today.md          # active daily plan (regenerated each morning)
├── week.md           # weekly overview
└── days/
    ├── 2026-05-05.md # archived
    ├── 2026-05-06.md
    └── 2026-05-07.md
```

## Features

### Pull (services → markdown)
- **`/plan-today`** — Morning. Pulls today's calendar events + Todoist tasks + carry-forwards from yesterday into a time-blocked plan.
- **`/plan-week`** — Weekly overview with day-by-day breakdown and priority-grouped tasks.

### Push (markdown → services)
- **`/sync-today`** — Push hand-edits in `today.md` back to Calendar + Todoist (idempotent — synced lines get `(todoist:<id>)` or `(gcal:<eventId>)` suffixes).
- **`/quick-add <task>`** — Natural-language Todoist task with full fields:
  - project (`#Work`), section (`/Backend`), labels (`@bug`)
  - subtasks (`under "<parent>"`)
  - due (`tomorrow 4pm`, `every Monday`), priority (`P1`–`P4`), duration (`for 30m`)
  - notes (`note: ...`)
  - reminders (`remind me 30m before` — Pro only)
  - location reminders (Pro only)
- **`/add-event <event>`** — Google Calendar event with location, attendees, recurrence, reminders, Meet/Zoom link.
- **`/create-project <name>`** — Todoist project with color, parent, view style, favorite.

### Two-way
- **`/review-day`** — End-of-day. Cross-references Todoist completions, asks for mood/energy/wins/blockers, **closes completed tasks in Todoist**, archives the day.

### Read-only
- **`/review-yesterday`** — Summary of yesterday's archive.
- **`/review-week`** — Aggregate stats: completion rate, mood trends, recurring carry-forwards.

## Platform compatibility

| Platform | Works? |
|----------|--------|
| Claude Code (CLI / VS Code / JetBrains) | ✅ Full |
| Claude Desktop | ❌ Plugin format not supported (use as MCP server — roadmap) |
| Claude.ai web | ⚠️ Calendar only (paste commands into a Project) |

See [`INTEGRATIONS.md`](./INTEGRATIONS.md#platform-compatibility) for details.

## Install

### Option 1: As a Claude Code plugin (recommended)

```bash
# Clone locally
git clone https://github.com/janam87/claude-daily-planner.git ~/.claude/plugins/daily-planner

# Then in Claude Code:
/plugin
```

Select `daily-planner` from the list to enable.

### Option 2: Manual

Copy the `commands/` and `templates/` directories into your project's `.claude/` directory.

## Setup

### 1. Set environment variables

Copy `.env.example` to `.env` (or export in your shell):

```bash
export TODOIST_API_TOKEN="your_token_here"
export DAILY_PLANNER_HOME="$HOME/journal"        # optional, defaults to $PWD
export DAILY_PLANNER_TIMEZONE="America/New_York" # optional, defaults to UTC
export DAILY_PLANNER_CALENDAR_ID="primary"       # optional
```

### 2. Wire up integrations

See [`INTEGRATIONS.md`](./INTEGRATIONS.md):
- **Todoist** — get an API token from your developer settings
- **Google Calendar** — enable the connector in Claude

### 3. Run your first day

```
/plan-today          # morning
/quick-add ...       # add things during the day
/sync-today          # push edits back
/review-day          # night
```

## Data ownership

Everything lives in plain markdown in `$DAILY_PLANNER_HOME`. Back it up to git, Obsidian, iCloud — whatever. The plugin never phones home.

## Adding more APIs

Want to add Linear, Jira, Notion, Apple Calendar, etc.? See [`INTEGRATIONS.md`](./INTEGRATIONS.md#adding-a-new-integration).

## Contributing

PRs welcome. The plugin is intentionally minimal — markdown commands, two templates, plain markdown output. Don't over-engineer it.

## License

MIT — see [`LICENSE`](./LICENSE).
