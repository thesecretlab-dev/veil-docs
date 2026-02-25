# FIND ME: VEILVM Agent Handoff

Last updated: 2026-02-20
Owner context: Josh / VEILVM

## Mission Tonight

1. Keep VEILVM chain running continuously.
2. Finish wiring VEIL Markets frontend to real VEIL execution flow.

## Critical Guardrails

- Use VEILVM paths only.
- Do **not** use old VEIL2 stack for this flow.
- Primary chain compose file:
  - `C:\Users\Josh\hypersdk\examples\veilvm\docker-compose.local.yml`

## Current Blocker

- Docker Desktop is stuck due WSL timeout:
  - `DockerDesktop/Wsl/CommandTimedOut`
  - `wsl.exe -l -v --all` times out
  - Docker API returns HTTP 500 on `dockerDesktopLinuxEngine`

## Fast Recovery Path (Post-Reboot)

1. Verify Docker:
   - `docker version`
2. Start VEILVM node:
   - `cd C:\Users\Josh\hypersdk\examples\veilvm`
   - `docker compose -f docker-compose.local.yml up -d --build node`
3. Readiness:
   - `http://127.0.0.1:9660/ext/health/readiness` should return `200`
4. Start VEIL order router:
   - `cd C:\Users\Josh\hypersdk\examples\veilvm`
   - `$env:ORDER_ROUTER_RELAY_SECRET='local-dev-relay'`
   - `go run ./cmd/veilvm-order-router`
5. Router health:
   - `http://127.0.0.1:9098/health` should return `200`
6. Start frontend:
   - `cd C:\Users\Josh\Desktop\private-github-ready-20260219\veil-frontend`
   - `$env:VEIL_ORDER_API_BASE='http://127.0.0.1:9098'`
   - `npm run dev`

## Key Handoff Docs

- `C:\Users\Josh\hypersdk\examples\veilvm\REBOOT_HANDOFF_2026-02-20.md`
- `C:\Users\Josh\hypersdk\examples\veilvm\README.md`
- `C:\Users\Josh\hypersdk\examples\veilvm\VEIL_MASTER_RUNBOOK.md`
- `C:\Users\Josh\hypersdk\examples\veilvm\VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`

## Frontend Repo (Active)

- `C:\Users\Josh\Desktop\private-github-ready-20260219\veil-frontend`

### Relevant files

- `app/api/orders/route.ts`
- `lib/veil-market-service.ts`
- `lib/market-api-client.ts`
- `components/trading-panel.tsx`
- `README.md`

### Important local behavior

- In dev, `/api/orders` now defaults to `http://127.0.0.1:9098` if `VEIL_ORDER_API_BASE` is unset.
- Frontend distinguishes:
  - VEIL-native markets
  - Polygon-native markets (with 0.03% routing fee)

## VEIL Order Router (VM side)

- Path: `C:\Users\Josh\hypersdk\examples\veilvm\cmd\veilvm-order-router`
- Default listen: `:9098`
- Endpoints:
  - `POST /orders`
  - `POST /evm/intents/execute`
  - `POST /evm/liquidity/execute`
  - `GET /health`

## Current Priority Checklist for New Agent

1. Recover Docker/WSL and get VEILVM node healthy on `9660`.
2. Start router on `9098`.
3. Verify frontend order submission returns VEIL tx hash path (not unconfigured).
4. Run one real end-to-end UI order flow and capture tx hash output.
5. Keep chain up while finishing remaining frontend polish.

## Known Workspace State

- Repos are dirty with many pre-existing changes.
- Do not reset/revert broad changes.
- Only make minimal targeted edits.
