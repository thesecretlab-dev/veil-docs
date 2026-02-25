# VEILVM Reboot Handoff (Feb 20, 2026)

This gets the local VEILVM chain and frontend back online after a Windows reboot.

## 0. Fast Restart Wizard (target: under 60s)

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File C:\Users\Josh\hypersdk\examples\veilvm\scripts\install-wizard.ps1
```

If fast mode misses readiness, run with automatic repair:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File C:\Users\Josh\hypersdk\examples\veilvm\scripts\install-wizard.ps1 -Repair
```

Troubleshooting details are tracked in:

- `C:\Users\Josh\hypersdk\examples\veilvm\RESTART_PATH_TROUBLESHOOTING_2026-02-20.md`

## 1. Start Docker and verify daemon

```powershell
$env:PATH='C:\Program Files\Docker\Docker\resources\bin;'+$env:PATH
docker version
```

If Docker still errors, reboot again before continuing.

## 2. Bring up VEILVM node (correct stack)

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm
docker compose -f docker-compose.local.yml up -d --build node
```

Readiness check:

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:9660/ext/health/readiness | Select-Object -ExpandProperty StatusCode
```

Expected: `200`.

## 3. Start VEIL order router

```powershell
$env:PATH='C:\Program Files\Docker\Docker\resources\bin;'+$env:PATH
docker rm -f veilvm-order-router
docker run -d --name veilvm-order-router `
  --add-host host.docker.internal:host-gateway `
  -p 127.0.0.1:9098:9098 `
  -e ORDER_ROUTER_RELAY_SECRET=local-dev-relay `
  -e ORDER_NODE_URL=http://host.docker.internal:9660 `
  -v C:\Users\Josh\hypersdk:/workspace `
  -w /workspace/examples/veilvm `
  golang:1.23-bullseye `
  bash -lc 'export PATH=/usr/local/go/bin:$PATH; go run ./cmd/veilvm-order-router'
```

Health check (new terminal):

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:9098/health | Select-Object -ExpandProperty StatusCode
```

Expected: `200`.

## 4. Start frontend wired to local router

```powershell
Set-Location C:\Users\Josh\Desktop\private-github-ready-20260219\veil-frontend
$env:VEIL_ORDER_API_BASE='http://127.0.0.1:9098'
$env:NEXT_PUBLIC_VEIL_TX_EXPLORER_BASE=''
npm run dev
```

Open `http://localhost:3000`.

## 5. Quick API sanity

```powershell
Invoke-WebRequest -UseBasicParsing http://localhost:3000/api/markets | Select-Object -ExpandProperty StatusCode
Invoke-WebRequest -UseBasicParsing http://localhost:3000/api/orders -Method Post -ContentType 'application/json' -Body '{"marketId":"test","side":"buy","outcome":"yes","amountUsd":1,"walletAddress":"0x123","nativeNetwork":"veil"}' | Select-Object -ExpandProperty StatusCode
```

Expected:

- `/api/markets` returns `200`
- `/api/orders` should no longer return `503 unconfigured` once router is up

## 6. Important path guard

Use only:

- `C:\Users\Josh\hypersdk\examples\veilvm\docker-compose.local.yml`

Do not use old deprecated stack paths for this flow.

