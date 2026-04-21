# deepflow-dashboard

Analytics dashboard for [Claude Code](https://claude.ai/code) — visualize token usage, costs, cache efficiency, and activity across sessions.

## Quick Start

```bash
npx deepflow-dashboard install   # wire telemetry hooks into ~/.claude/settings.json
npx deepflow-dashboard           # start dashboard at http://localhost:3333
```

## Modes

### Local (default)

Reads directly from your `~/.claude/` transcripts and quota history. Data is stored in a local SQLite database at `~/.claude/deepflow-dashboard.db`.

```bash
npx deepflow-dashboard local [--port 3333]
```

### Team Server

Runs a shared server that accepts telemetry from remote Claude Code instances.

```bash
npx deepflow-dashboard serve [--port 3333]
```

Push your local history to the team server:

```bash
npx deepflow-dashboard backfill --url http://your-team-server:3333
```

## CLI Reference

```
npx deepflow-dashboard              Start local dashboard (default)
npx deepflow-dashboard local        Start local dashboard
npx deepflow-dashboard serve        Start team server
npx deepflow-dashboard backfill     Backfill remote server with local data
npx deepflow-dashboard install      Install telemetry hooks
npx deepflow-dashboard uninstall    Remove telemetry hooks

Options:
  --port <n>      Port to listen on (env: DASHBOARD_PORT, default: 3333)
  --url <url>     Remote server URL (for backfill)
```

## Dashboard Views

| View | Description |
|------|-------------|
| Cost Overview | Total spend by model over time |
| Sessions | Per-session token and cost breakdown |
| Quota Status | Quota window usage and resets |
| Cache Efficiency | Cache hit rates and savings |
| Activity Heatmap | Usage patterns by hour/day |
| Token by Tool | Token consumption per Claude Code tool |
| Peak Hours | Busiest times of day |

## Development

```bash
npm install
npm run dev        # Vite dev server on :5173 (proxies /api → localhost:3333)
npm run build      # Build client + server to dist/
npm run typecheck  # Type check only
```

Requires Node.js 18+.

## How It Works

1. **Hooks** — Installed into `~/.claude/settings.json`, these capture Claude Code events (tool calls, session end, etc.) and write structured telemetry.
2. **Ingestion** — On startup, the server parses `~/.claude/` transcripts and quota history into a local SQLite database.
3. **API** — A Hono web server exposes `/api/*` endpoints (sessions, costs, quota, activity, cache, tools).
4. **Dashboard** — A React + Recharts SPA polls the API and renders the views.

In team mode, instances POST telemetry to `/api/ingest` instead of reading local files.
