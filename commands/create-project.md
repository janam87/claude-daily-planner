---
description: Create a Todoist project (with optional parent, color, view style)
argument-hint: <project name> [color] [under "<parent>"]
allowed-tools: Bash(curl:*), Read
---

# Create Project

Create a Todoist project. User input: `$ARGUMENTS`

## Configuration

- `TODOIST_API_TOKEN` — **required**

## Step 1: Parse input

Extract:
- **Name** (required)
- **Color** — one of: `berry_red`, `red`, `orange`, `yellow`, `olive_green`, `lime_green`, `green`, `mint_green`, `teal`, `sky_blue`, `light_blue`, `blue`, `grape`, `violet`, `lavender`, `magenta`, `salmon`, `charcoal`, `grey`, `taupe` (default: `charcoal`)
- **Parent**: if user said `under "<name>"`, look up parent project_id
- **View style**: `list` or `board` (default: `list`)
- **Favorite**: if user said `favorite` or `pin`, set `is_favorite: true`

Examples:
- `/create-project Side Hustle blue board favorite`
- `/create-project Q3 Goals under "Work"`
- `/create-project Books to Read green`

## Step 2: Resolve parent (if given)

```bash
curl -s -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  "https://api.todoist.com/api/v1/projects"
```

Match parent name case-insensitively. If multiple match, ask which one.

## Step 3: Create the project

```bash
curl -s -X POST "https://api.todoist.com/api/v1/projects" \
  -H "Authorization: Bearer $TODOIST_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Side Hustle",
    "color": "blue",
    "parent_id": null,
    "view_style": "board",
    "is_favorite": true
  }'
```

## Step 4: Confirm

Show:
- Project ID + name
- URL: `https://app.todoist.com/app/project/<id>`
- Parent (if any)
- Color, view style, favorite status

## Important

- Todoist API base: `https://api.todoist.com/api/v1`
- Free Todoist allows up to 5 active projects; Pro allows 300. If creation fails with quota error, surface it.
- Sub-projects (`parent_id`) are a Pro feature. If the user is on free, the parent will be ignored or rejected.
