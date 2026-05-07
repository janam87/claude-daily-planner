# Changelog

All notable changes to this project. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versioning: [SemVer](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] — 2026-05-07

### Added
- **Bidirectional sync.** Plugin now writes to Todoist + Google Calendar, not just reads.
- `/sync-today` — push hand-edited `today.md` back to Calendar + Todoist (idempotent via `(todoist:<id>)` / `(gcal:<eventId>)` tags).
- `/add-event` — create Google Calendar event with location, attendees, recurrence (RRULE), reminders, conferencing (Meet).
- `/create-project` — create Todoist project with color, parent, view style, favorite.
- `/quick-add` expanded to support full Todoist field set: subtasks (`parent_id`), labels, sections, duration, description (notes), reminders (Pro only — time + location).
- `/review-day` now closes completed tasks in Todoist (was: local-only checkbox update).
- `INTEGRATIONS.md` adds platform compatibility matrix (Claude Code / Desktop / web), Pro-only feature list, full read/write endpoint reference.
- `CONTRIBUTING.md`, `CHANGELOG.md`, GitHub Actions CI for secret scanning + JSON validation.

### Changed
- Default `DAILY_PLANNER_TIMEZONE` from `Asia/Kolkata` → `UTC`.
- Plugin manifest now includes `author`, `homepage`, `repository` fields.

### Security
- Stripped hardcoded Todoist token (`717f3355…`) from all commands.
- All API tokens now read from env vars (`TODOIST_API_TOKEN`).

## [0.1.0] — 2026-05-07

### Added
- Initial release. Six slash commands: `/plan-today`, `/plan-week`, `/review-day`, `/review-yesterday`, `/review-week`, `/quick-add`.
- Templates for daily + weekly markdown output.
- Read-only integrations: Todoist (curl) + Google Calendar (claude.ai connector).
- Plain markdown archive in `days/YYYY-MM-DD.md`.

[Unreleased]: https://github.com/janam87/claude-daily-planner/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/janam87/claude-daily-planner/releases/tag/v0.2.0
[0.1.0]: https://github.com/janam87/claude-daily-planner/releases/tag/v0.1.0
