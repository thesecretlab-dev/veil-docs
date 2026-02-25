# VEIL + ANIMA Mainnet Launch Handoff (2026-02-22)

Updated: 2026-02-22  
Scope: VEIL chain launch path plus ANIMA runtime launch readiness.

Primary execution root for next agent:
- `C:\Users\Josh\hypersdk\examples\veilvm`
- ANIMA (`C:\Users\Josh\Desktop\veil-automaton`) is only one launch gate (`G12`), not the primary lane.

## 1) Current Truth (Authoritative)

Primary source:
- `C:\Users\Josh\hypersdk\examples\veilvm\VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`

Current snapshot in checklist (Section 5):
- `G0..G12` are all `PASS` or `PASS (local)` as of 2026-02-22.
- `G12` is now `PASS (local)` with ANIMA readiness evidence at:
  - `evidence-bundles/anima-readiness/latest.txt` -> `anima-20260222-143834`
- Current decision is still explicitly: `NO-GO FOR PRODUCTION`.

Why no-go still stands:
- Evidence is currently local-scoped for multiple gates.
- Production sign-off sheet in checklist is not completed.
- Remaining production-operational closure still needs final operator custody and production-validator execution trail.

## 2) What Next Agent Must Achieve

Goal:
- Continue VEIL mainnet launch work across the full VEIL stack, with primary focus on chain/bridge/gate closure in `veilvm`.
- Treat ANIMA work as scoped support for `G12`, not as the main project.

Hard done condition:
- Checklist decision flips from `NO-GO` to `GO` only after production evidence and role sign-offs are complete.

Next exact step (first command to run):
```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm\scripts
npm run check:prelaunch
```

## 3) Exact Execution Order (Do This In Order)

### Step 0 - Refresh context and pointers
Run:
```powershell
Get-Content C:\Users\Josh\hypersdk\examples\veilvm\VEIL_PRODUCTION_LAUNCH_CHECKLIST.md
Get-Content C:\Users\Josh\hypersdk\examples\veilvm\LIVE_DEVLOG.md
Get-Content C:\Users\Josh\hypersdk\examples\veilvm\evidence-bundles\critical-phase\latest.txt
Get-Content C:\Users\Josh\hypersdk\examples\veilvm\evidence-bundles\anima-readiness\latest.txt
```

### Step 1 - Bring up VEIL chain and router (Docker-first)
From `C:\Users\Josh\hypersdk\examples\veilvm`:
```powershell
$env:PATH='C:\Program Files\Docker\Docker\resources\bin;'+$env:PATH
docker compose -f docker-compose.local.yml up -d --build node
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:9660/ext/health/readiness | Select-Object -ExpandProperty StatusCode
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
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:9098/health | Select-Object -ExpandProperty StatusCode
```

### Step 2 - Re-run critical gates
From `C:\Users\Josh\hypersdk\examples\veilvm\scripts`:
```powershell
npm run check:prelaunch
node run-critical-phase-gates.mjs
npm run launch:rehearsal
```
Pass condition:
- `check:prelaunch` reports `overallPass=true`.
- `critical-phase/latest.txt` points to a run with `overall_pass=true`.

### Step 3 - Re-validate companion bridge and opaque relay path
From `C:\Users\Josh\hypersdk\examples\veilvm\scripts`:
```powershell
npm run bridge:check-mainnet-live
npm run relay:opaque -- --from-block 94
```
Notes:
- `bridge:check-mainnet-live` fails closed if `MAINNET_RELAYER_PRIVATE_KEY` is missing.
- Opaque relay requires executor credentials and relay secret alignment.

### Step 4 - ANIMA readiness re-check (chain-aligned)
From `C:\Users\Josh\Desktop\veil-automaton`:
```powershell
cmd /c pnpm --filter @veil/vm-sdk test -- --run
cmd /c pnpm --filter @veil/vm-sdk build
cmd /c pnpm --filter @veil/anima build
```
Pass condition:
- VM SDK and ANIMA builds/tests remain green against current strict-private runtime expectations.

### Step 5 - Production conversion of local-pass gates
Execute production closure work and archive evidence for:
- `G2`: production-validator threshold key ceremony + rollout audit + adversarial no-fallback proof.
- `G4/G5`: production parameter freeze and treasury/risk evidence snapshots.
- `G6`: production relay round-trip evidence with live credentials and finalized execution proofs.
- `G12`: production-safe ANIMA runtime evidence set with launch-copy parity and no overclaims.

### Step 6 - Final decision packet
Before any GO call:
- Update checklist statuses and evidence pointers.
- Complete checklist sign-off sheet for all owner roles.
- Re-run launch rehearsal packet and confirm signature checks.

## 4) VEIL Chain + ANIMA Integration Notes

- VEIL chain ID used by SDK path remains `22207`.
- ANIMA strict-private Tier 0 action path remains:
  - `2 CommitOrder`
  - `3 RevealBatch`
  - `17 SubmitBatchProof`
  - `4 ClearBatch`
  - plus admin ops `18` and `41` where required.
- New SDK requirement in current `veil-automaton` working tree:
  - `registerIdentity()` now requires ERC-8004 passport registration support (`passportRegistrar`) before `zeroid_register` can proceed.
  - File: `C:\Users\Josh\Desktop\veil-automaton\sdks\veil-vm\src\client.ts`

## 5) Known Operational Failure Modes

- PowerShell execution policy blocks direct `pnpm` on this host.
  - Use `cmd /c pnpm ...`.
- Missing relayer key env var blocks bridge live checks.
- Missing hardened-owner custody blocks some post-rotation stateful operations.
- Legacy non-opaque envelopes are rejected by hardened relay path (`ENVELOPE_PRIVACY_REJECTED` expected).

## 6) Files Next Agent Must Keep In Sync

- `C:\Users\Josh\hypersdk\examples\veilvm\VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
- `C:\Users\Josh\hypersdk\examples\veilvm\LIVE_DEVLOG.md`
- `C:\Users\Josh\hypersdk\examples\veilvm\evidence-bundles\critical-phase\latest.txt`
- `C:\Users\Josh\hypersdk\examples\veilvm\evidence-bundles\anima-readiness\latest.txt`
- `C:\Users\Josh\Desktop\private-github-ready-20260219\veil-frontend\docs\claude-handshake\README.md`
- `C:\Users\Josh\Desktop\private-github-ready-20260219\veil-frontend\docs\claude-handshake\surface-translation-matrix.md`
- `C:\Users\Josh\Desktop\private-github-ready-20260219\veil-frontend\docs\claude-handshake\surface-translation-registry.json`

## 7) Session Resume

- Continuance token (VEIL program, current session):
  - `VEIL-PROGRAM-20260222T203659Z-B18E4982`
- Continuance token file:
  - `C:\Users\Josh\Desktop\VEIL_CONTINUANCE_TOKEN_2026-02-22.txt`
- Legacy automaton-scoped token (reference only):
  - `VEIL-AUTOMATON-20260222T203019Z-657471E2`
- Codex resume key:
  - `019c8456-1bd8-7490-9b1d-dfd4d6c44526`
- Resume command:
  - `codex resume 019c8456-1bd8-7490-9b1d-dfd4d6c44526`
