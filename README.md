# Peak Sales Momentum Dashboard

A static HTML dashboard showing Michelle Anderson's weekly sales activity for The Peak, driven by a JSON file the weekly cron job produces.

## Viewing locally

```bash
cd "Peak Dashboard"
python -m http.server 8000
# open http://localhost:8000
```

> **Why a server?** `index.html` fetches `data/latest.json` via `fetch()`. Browsers block `fetch()` from `file://` URLs, so you need any HTTP server — Python's is the simplest.

## How the data flow works

```
Every Monday morning
  └─ Claude Code skill: sales-momentum-weekly
        ├─ Pulls last 7 days from HubSpot (emails, deals, meetings)
        ├─ Classifies activity against the H2 plan framework
        ├─ Appends current week to data/history.json
        ├─ Overwrites data/latest.json with the new snapshot
        └─ Pushes to main → GitHub Actions deploys automatically
```

**`data/latest.json`** — always the current week. `index.html` reads this on load.

**`data/history.json`** — array of past weekly snapshots, oldest first. The dashboard uses the last 4 entries to draw the trend bar chart. The cron job appends to this file before overwriting `latest.json`.

## Updating data manually

1. Edit `data/latest.json` with the new week's numbers (follow the existing JSON schema).
2. Append the previous week's snapshot to `data/history.json`:
   ```json
   [
     { "week_start": "May 19, 2026", "proposals_this_week": 4, "proposals_weekly_target": 5, "pace_status": "slightly_behind" },
     ...
   ]
   ```
3. Commit and push to `main`.

## Deploying

Push to `main`. The GitHub Actions workflow (`.github/workflows/pages.yml`) runs automatically and deploys to GitHub Pages.

**First-time setup:**
1. Go to your repo → Settings → Pages
2. Set **Source** to `GitHub Actions`
3. Push to `main` — the workflow will handle the rest

The live URL will be `https://<your-org>.github.io/<repo-name>/`.

## JSON schema reference

See `data/latest.json` for the full schema. Key fields:

| Field | Type | Description |
|-------|------|-------------|
| `topline.proposals_this_week` | number | Proposals that entered Proposal stage this week |
| `topline.proposals_ytd` | number | YTD total |
| `topline.cold_emails` | number | True cold outreach count (floor: 5/week) |
| `topline.nurture_emails` | number | All warm outbound (active pipeline, EL, L+L, etc.) |
| `topline.pace_status` | string | `on_track` / `slightly_behind` / `behind` |
| `renewal_watchlist` | array | Per-account status with `status_color`: `good` / `warn` / `danger` |
| `drilldowns` | object | Click-through detail for each topline tile (keys: `proposals`, `ytd`, `pipeline_dollars`, `meetings`, `cold`, `nurture`) |

## Files

```
/
├── index.html              # The dashboard
├── data/
│   ├── latest.json         # Current week (overwritten weekly by cron)
│   └── history.json        # Past weeks (appended weekly by cron)
├── .github/
│   └── workflows/
│       └── pages.yml       # GitHub Pages deploy workflow
└── README.md               # This file
```
