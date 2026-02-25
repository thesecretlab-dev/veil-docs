# VEILVM Restart Path Troubleshooting (2026-02-20)

## Scope

This documents the exact restart failures hit on Feb 20, 2026 and the recovery actions that worked.

## Failures Observed

1. Docker daemon unavailable after reboot.
   - Symptom: `docker version` failed with `open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified.`
   - Recovery: launch Docker Desktop, then wait until `docker version` shows both Client and Server.

2. VEILVM node container started, but readiness stayed `503`.
   - Symptom: `http://127.0.0.1:9660/ext/health/readiness` never reached `200`.
   - Logs showed tracked subnet bootstrap failure and chain creation errors, including:
     - `subnets not bootstrapped`
     - `error while creating new snowman vm rpc error: ... Compact start is not less than end`
   - Root cause: stale/corrupted tracked-subnet state in local DB for the previously configured subnet ID.

3. `scripts/setup-local.mjs` initially failed.
   - Symptom: `Node did not become healthy in 120s`.
   - Root cause: script waits on `/ext/health` and could not proceed while stale tracked subnet kept node unhealthy.

4. Router startup from Windows host (`go run ./cmd/veilvm-order-router`) failed.
   - Symptom: build errors on Windows toolchain/deps (`github.com/DataDog/zstd`, `blst` binding errors).
   - Root cause: local host build path is not reliable in this Windows environment for this module graph.

5. Router startup in Docker failed twice before final fix.
   - Failure A: `go: command not found`.
     - Cause: container shell PATH did not include `/usr/local/go/bin`.
   - Failure B: `replace ../../` resolution errors (`reading /go.mod: open /go.mod: no such file or directory`).
     - Cause: mounted only `examples/veilvm`; repo expects `../../` (hypersdk root).

6. EVMBench passthrough mode failed in multiple stages.
   - Symptom A: worker used `gpt-5.2-2025-12-11` and failed with invalid model ID on OpenRouter.
   - Root cause A: stale `evmbench/worker` image still contained old `worker_runner/model_map.json`.
   - Symptom B: after model mapping fix, second `/v1/responses` request failed with `invalid_prompt` / `invalid_union`.
   - Root cause B: OpenRouter rejected some `input` item shapes emitted by Codex Responses API flow.
   - Recovery:
     - force worker/backend rebuild via `scripts/evmbench-up.ps1 -UseSigmaOpenRouterPassthrough`
     - normalize Responses API payloads in `oai_proxy/routers/catch_all.py` (flatten nested input arrays, sanitize null/typed fields, drop unsupported item types)
     - rebuild backend image and restart proxy profile.

## Corrected Recovery Path

### Preferred entrypoint

Use the restart wizard first:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File C:\Users\Josh\hypersdk\examples\veilvm\scripts\install-wizard.ps1
```

Use `-Repair` if fast path does not reach readiness in time.

### 1) Bring Docker up

```powershell
$env:PATH='C:\Program Files\Docker\Docker\resources\bin;'+$env:PATH
docker version
```

### 2) If readiness is stuck `503` with tracked subnet errors

1. Temporarily remove `--track-subnets=...` from `docker-compose.local.yml`.
2. Restart node:

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm
docker compose -f docker-compose.local.yml up -d --build node
```

3. Wait for both:
   - `http://127.0.0.1:9660/ext/health` -> healthy
   - `http://127.0.0.1:9660/ext/health/readiness` -> `200`

4. Recreate subnet+chain:

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm\scripts
node setup-local.mjs
```

5. Put returned subnet ID back into compose as `--track-subnets=<newSubnetID>`.
6. Restart node again with compose.

### 3) Router launch (reliable path in this environment)

Run router in Linux container with hypersdk root mounted:

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

### 4) EVMBench passthrough recovery (OpenRouter)

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm\scripts
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-up.ps1 -UseSigmaOpenRouterPassthrough
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-audit.ps1
```

Expected success evidence:

- `evidence-bundles/audit-closure/evmbench-20260220-133630/job-status.json` -> `status: succeeded`
- `evidence-bundles/audit-closure/evmbench-20260220-133630/summary.md` -> `findings: 0`

## IDs Produced During This Recovery

- Subnet ID: `4juHwsyQKbKo6hgwHgd2Xy3gRuEnuDHHT2NZhCKf66fWeCkMx`
- Chain ID: `2CdK3iHBweFSZhh5XBgLYDaC2U7SoyqEzDaTRhmMFwSLLCm1Xb`

`docker-compose.local.yml` was updated to:

- `--track-subnets=4juHwsyQKbKo6hgwHgd2Xy3gRuEnuDHHT2NZhCKf66fWeCkMx`

## Recommended Follow-Up Fixes

1. Add a first-party restart script that automates:
   - Docker check
   - health/readiness checks
   - fallback `setup-local.mjs`
   - compose track-subnet update
2. Add a documented router startup path for Windows users (containerized path above).
3. Add a preflight that fails fast when `--track-subnets` points at a broken subnet state and prints the repair path.

