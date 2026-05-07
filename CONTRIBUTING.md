# Contributing

Thanks for considering a contribution. The plugin is intentionally minimal — please keep PRs focused.

## Quick start

```bash
git clone https://github.com/janam87/claude-daily-planner.git
cd claude-daily-planner
cp .env.example .env  # fill in TODOIST_API_TOKEN
```

Test locally by pointing Claude Code at this directory:

```
/plugin
```

…and select `daily-planner`.

## Project structure

```
.claude-plugin/plugin.json   # manifest
commands/                    # slash commands (each *.md is one command)
templates/                   # daily.md, weekly.md output templates
CLAUDE.md                    # auto-loaded conventions + API reference
INTEGRATIONS.md              # API setup + how to add new integrations
```

## Adding a new command

1. Create `commands/<name>.md` with frontmatter:
   ```yaml
   ---
   description: short one-liner
   argument-hint: <args if any>
   allowed-tools: Bash(curl:*), Read, Write
   ---
   ```
2. Body = step-by-step instructions Claude follows. Use markdown headers (`## Step 1: ...`).
3. Reference env vars (`$TODOIST_API_TOKEN`, `$DAILY_PLANNER_HOME`, etc.) — never hardcode.
4. Reference templates via `${CLAUDE_PLUGIN_ROOT}/templates/...`.

## Adding a new integration

See [`INTEGRATIONS.md#adding-a-new-integration`](./INTEGRATIONS.md#adding-a-new-integration) for the 7-step recipe.

## Coding rules

- **No secrets in commits.** CI runs gitleaks. Use `$ENV_VAR` for tokens.
- **No hardcoded user data** (timezones default to `UTC`, calendar to `primary`).
- **Idempotency**: any push command must tag created items with `(todoist:<id>)` or `(gcal:<eventId>)` so re-running doesn't duplicate.
- **Confirm before destructive ops** (delete, overwrite). Use `AskUserQuestion`.
- **Bidirectional sync conflicts**: default to local-wins, ask the user.

## PR checklist

- [ ] No secrets / personal info added
- [ ] Updated `CLAUDE.md` if new env vars or APIs
- [ ] Updated `INTEGRATIONS.md` if new integration
- [ ] Updated `README.md` feature list if user-facing
- [ ] Added entry to `CHANGELOG.md` under `## [Unreleased]`
- [ ] Frontmatter on new commands has correct `allowed-tools`
- [ ] Tested in Claude Code locally

## Reporting issues

Include:
- Claude Code version (`claude --version`)
- Command that misbehaved (`/plan-today`, etc.)
- Relevant env vars (redact tokens)
- Output / error message

## License

By contributing you agree your work is licensed under MIT (same as the project).
