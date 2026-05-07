# Integrations

The plugin ships with two integrations: **Todoist** (required, REST API) and **Google Calendar** (optional, claude.ai connector). Both are **bidirectional** — read and write.

---

## Platform compatibility

| Platform | Plugin install | Why |
|----------|---------------|-----|
| **Claude Code (CLI / VS Code / JetBrains)** | ✅ Yes | Plugin format (`.claude-plugin/`) is Claude Code's native system. Bash + curl available. |
| **Claude Desktop** | ❌ Not as plugin | Desktop uses MCP servers, not plugins. The Google Calendar connector works in Desktop, but Todoist needs Bash/curl which Desktop doesn't expose. |
| **Claude.ai (web)** | ⚠️ Partial — as a Project | Paste the command bodies as Project custom instructions. Calendar works (connector). Todoist won't (no shell). |

To support Claude Desktop natively, the plugin would need to be ported to an **MCP server** that wraps the Todoist API in tool calls. Roadmap item — PR welcome.

---

## Todoist (required)

### 1. Get a token

1. Open https://app.todoist.com/app/settings/integrations/developer
2. Copy your **API token** (40 hex chars).

### 2. Export it

Add to your shell rc (`~/.zshrc`, `~/.bashrc`):

```bash
export TODOIST_API_TOKEN="paste_token_here"
```

Reload: `source ~/.zshrc`.

### 3. Verify

```bash
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/tasks?filter=today" | head
```

Should return JSON.

### Why v1, not v2?

The Todoist npm package (`@doist/todoist-mcp`) targets the deprecated v2 API. We call v1 directly via `curl` so the plugin keeps working.

### Pro-only features

| Feature | Free | Pro |
|---------|------|-----|
| Tasks, projects, labels, sections, subtasks, due dates, priorities | ✅ | ✅ |
| Notes/description, duration | ✅ | ✅ |
| Reminders (time-based) | ❌ | ✅ |
| Reminders (location-based) | ❌ | ✅ |
| Sub-projects (nested) | ❌ | ✅ |
| > 5 active projects | ❌ | ✅ |

If you're on free, reminder calls return 200 but no reminder fires. The plugin warns when you ask for one.

---

## Google Calendar (optional)

The plugin uses Claude's built-in **Google Calendar connector**, not a separate API token. Bidirectional out of the box.

### 1. Enable connector

1. Open Claude (web or desktop).
2. Settings → Connectors → **Google Calendar** → Connect.
3. Authorize the OAuth scope (read **and** write).

### 2. Verify in Claude Code

Run `/plan-today` — it calls `mcp__claude_ai_Google_Calendar__list_events`. If the connector isn't connected, the command warns and skips Calendar steps (Todoist still works).

### 3. Choose a calendar

Default = your primary. To target a specific one:

```bash
export DAILY_PLANNER_CALENDAR_ID="work@yourcompany.com"
```

Find IDs in Google Calendar → Settings → Integrate calendar.

### Tools used

| Tool | Direction |
|------|-----------|
| `mcp__claude_ai_Google_Calendar__list_events` | read |
| `mcp__claude_ai_Google_Calendar__get_event` | read |
| `mcp__claude_ai_Google_Calendar__list_calendars` | read |
| `mcp__claude_ai_Google_Calendar__suggest_time` | read |
| `mcp__claude_ai_Google_Calendar__create_event` | write |
| `mcp__claude_ai_Google_Calendar__update_event` | write |
| `mcp__claude_ai_Google_Calendar__delete_event` | write |
| `mcp__claude_ai_Google_Calendar__respond_to_event` | write |

Recurrence uses RFC 5545 RRULE strings (`RRULE:FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR`). Conferencing (`hangoutsMeet`) is supported via `conferenceData.createRequest`.

---

## Adding a new integration

The plugin is just slash commands written in markdown — adding integrations means editing them. Pattern:

### 1. Pick an env var name

Convention: `<SERVICE>_API_TOKEN` (e.g. `LINEAR_API_TOKEN`, `NOTION_API_TOKEN`).

### 2. Document it in `CLAUDE.md`

Add a row to the env vars table. This file is auto-loaded by Claude when the plugin is active, so the model "knows" about your new var.

### 3. Add an API section in `CLAUDE.md`

Include both **read** and **write** endpoints. Example:

```markdown
## Linear API

- Base: `https://api.linear.app/graphql`
- Auth: `Authorization: $LINEAR_API_TOKEN`
- Docs: https://developers.linear.app/docs

### Read
GraphQL `query { issues { ... } }`

### Write
GraphQL `mutation { issueCreate(input: {...}) { ... } }`
```

### 4. Add a fetch step in commands that need it

Open the relevant command (e.g. `commands/plan-today.md`) and add a step:

```markdown
## Step 4b: Fetch Linear issues

curl -s -X POST "https://api.linear.app/graphql" \
  -H "Authorization: $LINEAR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query": "{ issues(filter: {assignee: {isMe: {eq: true}}}) { nodes { id title } } }"}'
```

### 5. Add a write command (if applicable)

Create `commands/<service>-add.md` with the same frontmatter pattern as `quick-add.md`.

### 6. Add `Bash(curl:*)` to `allowed-tools`

If the command's frontmatter doesn't already include it, add it so Claude can run the call without prompting.

### 7. Update README + .env.example

List the new integration in the README and add the env var to `.env.example`.

That's it — no code, no rebuild. Claude reads the markdown and follows your steps.

---

## Available connectors (Claude built-ins)

If the API has a Claude connector, prefer it over a raw token — auth is handled for you. Common ones:

- Google Calendar → `mcp__claude_ai_Google_Calendar__*`
- Gmail → `mcp__claude_ai_Gmail__*`
- Slack → `mcp__claude_ai_Slack__*`
- Google Drive → `mcp__claude_ai_Google_Drive__*`
- HubSpot → `mcp__claude_ai_HubSpot__*`
- Supabase → `mcp__claude_ai_Supabase__*`

Connectors expose both read and write. Check Settings → Connectors in Claude for the full list.

---

## Troubleshooting

**`TODOIST_API_TOKEN is not set`** — Run `echo $TODOIST_API_TOKEN`. If empty, re-source your shell rc or restart Claude Code.

**`401 Unauthorized` from Todoist** — Token invalid or revoked. Generate a new one at the URL above.

**Reminders silently no-op** — You're on Todoist free. Upgrade to Pro or remove the reminder phrase from your input.

**Google Calendar "tool not found"** — Connector not enabled. Re-check Settings → Connectors.

**Calendar event created but no Meet link** — Connector OAuth scope missing `https://www.googleapis.com/auth/calendar.events`. Disconnect and re-connect to refresh scopes.

**`429 Too Many Requests`** — Todoist rate limit. Wait 5s and retry; bulk syncs auto-back-off.

**Wrong timezone** — Set `DAILY_PLANNER_TIMEZONE` to a valid IANA zone (e.g. `America/Los_Angeles`, `Europe/London`).

**Items keep duplicating on `/sync-today`** — Lines need `(todoist:<id>)` or `(gcal:<eventId>)` suffix to be considered synced. Run `/sync-today` once and the suffixes get added automatically.
