# VEIL Standalone RPC Runbook

Updated: 2026-02-23

## Goal

Expose the self-host VEIL node RPC (`http://127.0.0.1:9660`) to a cloud-accessible HTTPS endpoint for remote validation flows (for example Build Games MVP `M4_ANIMA_VALIDATE_VEIL`).

## Current Live Endpoint (2026-02-23)

- Active standalone URL: `https://veil-rpc.thesecretlab.app`
- Health probe URL: `https://veil-rpc.thesecretlab.app/ext/health/readiness`
- Tunnel ID: `a15d2868-ffa5-456f-8109-c7c0baa584d0`

If you want this under VEIL website DNS (`rpc.veil.markets`), add this CNAME in the `veil.markets` DNS zone:

- `rpc.veil.markets` -> `a15d2868-ffa5-456f-8109-c7c0baa584d0.cfargotunnel.com`

## Scope

- Runtime source node: `<local-dev-path>` (`docker-compose.local.yml`)
- Local RPC: `http://127.0.0.1:9660`
- Tunnel transport: `cloudflared` (no deprecated VEIL2 surfaces)

## Preconditions

1. Local VEIL node is healthy.
2. `cloudflared` is installed on operator host.
3. Operator understands this endpoint is public-facing and should be rotated/disabled when not needed.

## 1) Verify local RPC health

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:9660/ext/health/readiness | Select-Object -ExpandProperty StatusCode
```

Expected: `200`

## 2) Start standalone RPC tunnel (quick mode)

Important: VEIL node allows specific HTTP host headers by default. Keep `--http-host-header 127.0.0.1` so origin requests are accepted.

```powershell
$logDir = "<local-dev-path>"
New-Item -ItemType Directory -Force -Path $logDir | Out-Null

cloudflared tunnel `
  --url http://127.0.0.1:9660 `
  --http-host-header 127.0.0.1 `
  --metrics 127.0.0.1:40513 `
  --logfile "$logDir\cloudflared.log" `
  --pidfile "$logDir\cloudflared.pid"
```

When started, capture the generated HTTPS URL (usually `https://<random>.trycloudflare.com`).

## 3) Verify remote health endpoint

```powershell
$remote = "https://REPLACE_WITH_TUNNEL_URL"
Invoke-WebRequest -UseBasicParsing "$remote/ext/health/readiness" | Select-Object -ExpandProperty StatusCode
```

Expected: `200`

## 4) Wire remote RPC into MVP flow

Pass the remote URL to Build Games MVP as `--veil-rpc-url-remote`:

```powershell
pnpm.cmd exec tsx src/index.ts --build-games-mvp `
  --payment-tx 0xYOUR_TX_HASH `
  --veil-rpc-url-remote https://YOUR_STANDALONE_RPC_URL `
  --veil-rpc-url-local http://127.0.0.1:9660
```

This closes the cloud-side VEIL probe dependency in `M4_ANIMA_VALIDATE_VEIL`.

## 5) Shutdown and rotate

```powershell
$pidFile = "<local-dev-path>"
if (Test-Path $pidFile) {
  $pid = Get-Content $pidFile -Raw
  Stop-Process -Id ([int]$pid) -Force
}
```

After shutdown, do not reuse stale tunnel URLs.

## Notes

- Current local compose binds RPC on loopback only (`127.0.0.1:9660:9650`), which is correct for host hardening.
- Tunneling is preferred over opening host firewall inbound rules.
- If you need a persistent hostname (`rpc.veil.markets`), use a named Cloudflare tunnel and DNS route, but keep origin host header handling equivalent.

