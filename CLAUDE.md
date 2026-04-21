# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Vite dev server on port 5173 (proxies /api to localhost:3333)
npm run build      # Vite build + tsc + copy assets to dist/
npm run typecheck  # Type check without emitting
```

No test runner is configured. Test files in `src/**/__tests__/` and `test/` are integration/unit tests run manually.

To run the dashboard locally after building:
```bash
node dist/server.js local   # or: npx deepflow-dashboard local
```

## Architecture

This is a **full-stack TypeScript app** — a Hono (Node.js) backend serving a React + Vite frontend — packaged as an npm CLI tool that visualizes Claude Code telemetry.

### Modes

The server runs in two modes (same binary, different code paths):

- **Local mode** (`local` subcommand): Ingests data from `~/.claude/` transcripts and quota history, stores in SQLite at `~/.claude/deepflow-dashboard.db`, serves the dashboard on port 3333.
- **Team mode** (`serve` subcommand): Skips ingestion, accepts POSTs at `/api/ingest` from remote Claude Code instances. A `backfill` subcommand pushes local data to a team server.

### Data Flow (Local Mode)

```
~/.claude/ transcripts
    → src/ingest/parsers/   (sessions, tokens, cache, quota, tools)
    → src/db/               (sql.js SQLite, migrations in schema.sql)
    → src/api/              (Hono routers: /api/sessions, /api/costs, /api/quota, etc.)
    → React client          (polls /api/* endpoints)
```

### Key Directories

- **`src/api/`** — Hono routers, one file per domain. `index.ts` composes them. All support `?user=` query param in team mode.
- **`src/ingest/parsers/`** — Independent parsers for each data type (history, sessions, tokens, cache, quota, tool-usage).
- **`src/db/`** — `index.ts` wraps sql.js; `schema.sql` defines 5 tables. Migrations run on startup.
- **`src/lib/`** — Shared utilities. `quota-window-parser.ts` has tests.
- **`src/client/`** — React app (Vite root). `DashboardContext.tsx` holds mode, selected user, and refresh interval. `hooks/useApi.ts` auto-appends `?user=` in team mode. Charts use Recharts wrappers in `components/charts/`.
- **`bin/cli.js`** — npm bin entry point; routes subcommands to `dist/server.js`.
- **`src/hooks/`** — Shell hook scripts installed into `~/.claude/settings.json` to capture telemetry.

### Cost Calculation

Pricing is fetched from the Anthropic pricing API at startup, falling back to `src/data/pricing-fallback.json`. Per-session cost = `(tokens_in × rate_input) + (tokens_out × rate_output) + (cache_write × rate_write) + (cache_read × rate_read)`.

### Frontend State

React Context (`DashboardContext`) is the only shared state. No Redux or Zustand. Components fetch data directly via `useApi` hook which wraps `fetch` and handles polling via `usePolling`.

## Build Output

- `dist/client/` — Vite-built frontend assets (served as static files by Hono)
- `dist/` — tsc-compiled server code
- `dist/db/schema.sql` — copied from `src/db/`
- `dist/data/pricing-fallback.json` — copied from `src/data/`

The copy step in `npm run build` is a shell command in the build script — if adding new static assets needed at runtime, add them there.
