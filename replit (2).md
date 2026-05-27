# WhatsApp Bot

A WhatsApp bot built with Baileys + Express. Supports image search, translation, summarization, reminders, economy, games, gangs, missions, achievements, and more — with a web QR login page and real-time dashboard.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/dashboard run dev` — run the dashboard (port 23183)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- QR login page: `/api/qr/page`
- Dashboard: `/`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- WhatsApp: @whiskeysockets/baileys 7.0.0-rc13
- AI: OpenAI (gpt-4o-mini) for translation, summarization, and Q&A
- Images: Pixabay API (hardcoded key in pixabay.ts)
- Build: esbuild (CJS bundle)
- Dashboard: React + Vite, real-time via WebSocket at /api/ws

## Where things live

- Bot entry: `artifacts/api-server/src/bot/index.ts`
- Commands: `artifacts/api-server/src/bot/commands/`
- Services: `artifacts/api-server/src/bot/services/`
- Shared state (QR, reminders, games): `artifacts/api-server/src/bot/state.ts`
- User database (JSON file): `artifacts/api-server/data/users.json` (auto-created)
- Group whitelist: `artifacts/api-server/data/groups.json`
- QR login page route: `artifacts/api-server/src/routes/qr.ts`
- Session files saved at: `artifacts/api-server/sessions/` (auto-created by Baileys)
- Dashboard frontend: `artifacts/dashboard/src/`

## Architecture decisions

- Baileys is externalized from the esbuild bundle (uses native deps) — loaded from node_modules at runtime
- Session is persisted to `sessions/` directory using Baileys `useMultiFileAuthState`
- User data is stored as a flat JSON file (`data/users.json`) — no PostgreSQL needed for this bot
- Reminders are in-memory (lost on restart)
- QR code delivered to dashboard in real-time via WebSocket (`/api/ws`) — no polling needed
- OpenAI client uses lazy initialization — server starts without OPENAI_API_KEY; AI commands fail gracefully when key is absent
- Reconnect guard (`_isConnecting` flag) prevents stacked reconnect loops
- Event scheduler (`_eventSchedulerStarted` flag) ensures timers fire only once per process

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Delete `artifacts/api-server/sessions/` to force re-pairing with a new WhatsApp account
- Baileys requires `@hapi/boom` as a peer dep — both are in dependencies
- `@whiskeysockets/baileys` and `protobufjs` need build scripts approved — added to `onlyBuiltDependencies` in `pnpm-workspace.yaml`
- OpenAI commands (.ترجم, .ملخص, .اسأل) need `OPENAI_API_KEY` env var to work
- Images use a hardcoded Pixabay API key in `src/bot/services/pixabay.ts`
- On 401 (loggedOut), the bot stops reconnecting — delete `sessions/` and restart to re-pair

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
