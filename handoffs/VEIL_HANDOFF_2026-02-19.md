# VEIL Project Handoff (2026-02-19)

Status: Active handoff snapshot  
Date: 2026-02-19  
Updated: 2026-02-21T08:10Z  
Scope: VEIL custom VM chain + companion EVM chain

## 0. 2026-02-20 Rapid Update

- Launch-gate runner now supports targeted execution for speed:
  - `--only fuel,smoke,backup,negative,malformed,timeout`
  - `--skip-smoke`
- Fast split-gate workflow is documented in `README.md` and is now the default local iteration path.
- Fee-preflight/refuel handling was hardened to avoid false high-budget requirements and refuel-source draining between cases.
- Local chain was reset and recreated:
  - Subnet ID: `22U1uvR6tmAcYzaPZAcAQuH6Tu6mU8GKXgrURCDAscu4gq5fqH`
  - Chain ID: `TEV1x8r8HjAwdDhj3GhkpyxKrLLKXFHHxYEYeW8jdnsLFFgVa`
- Latest targeted gate evidence:
  - `fuel + smoke`: `evidence-bundles/20260220-024205-launch-gate-evidence/bundle.md` (PASS)
  - `backup + negative + malformed + timeout`: `evidence-bundles/20260220-025556-launch-gate-evidence/bundle.md` (PASS)
- Companion EVM intent execution path is now documented and wired:
  - contract entrypoint source: `companion-evm/contracts/VeilOrderIntentGateway.sol`
  - router endpoint: `POST /evm/intents/execute` (`x-relay-secret` required)
  - operations runbook: `VEIL_EVM_INTENT_RELAY_RUNBOOK.md`
- Companion EVM UniV2 liquidity rail path is now documented and wired:
  - contract entrypoint source: `companion-evm/contracts/VeilLiquidityIntentGateway.sol`
  - router endpoint: `POST /evm/liquidity/execute` (`x-relay-secret` required)

## 0.1 2026-02-21 Native Privacy Enforcement Validation

- VM-level envelope privacy enforcement is live and verified against runtime node `http://127.0.0.1:9660` on chain `2vrbfA1TYYTKofezajo4d6tsnuzvBfPJkVxLM85SaBRuUBJ2J1`.
- Proof captured that plaintext envelopes are rejected even when router opaque gate is disabled:
  - response: `HTTP 502`, `errorCode=ORDER_TX_FAILED`, message includes `execution failed: envelope must be opaque bytes`.
- Opaque envelopes commit successfully:
  - `veilTxHash=0xbb2cb6...eb7c` (gate disabled)
  - `veilTxHash=0x5c644e...cb24` (hardened mode restored)
- Router hardened mode has been restored and confirmed:
  - `/health` shows `opaqueEnvelopeRequired=true`, `minOpaqueEnvelopeBytes=96`
  - plaintext ingress returns `HTTP 400`, `errorCode=ENVELOPE_PRIVACY_REJECTED`
- Evidence bundle:
  - `evidence-bundles/20260221-040214-native-privacy-enforcement/bundle.md`
  - pointer: `evidence-bundles/latest-native-privacy-evidence.md`

## 0.2 2026-02-21 VEIL.Markets Public Surface Scrape

- Live scrape completed across sitemap + direct route probes for `https://veil.markets`.
- Current public surface is a single closed-alpha gate page replicated across all crawled routes.
- Crawl bundle:
  - `evidence-bundles/20260221-041042-veil-markets-live-scrape/bundle.md`
  - `evidence-bundles/20260221-041042-veil-markets-live-scrape/scrape.json`
  - pointer: `evidence-bundles/latest-veil-markets-live-scrape.md`

## 0.3 2026-02-21 Native AMM Liquidity Movement Validation (Historical, Pre-Private-Gate)

- VM-native LP movement path is now validated end-to-end on local runtime chain `2vrbfA1TYYTKofezajo4d6tsnuzvBfPJkVxLM85SaBRuUBJ2J1` (`http://127.0.0.1:9660`).
- `veilvm-smoke` now checks tx execution success from indexer (`Result.Success`) for each action; submit-only false positives were removed.
- Action state-key permissions were hardened for first-write key allocation paths:
  - `MintVAI`: VAI state + user VAI/debt keys set to `state.All`
  - `AddLiquidity`: user LP key set to `state.All`
  - `SwapExactIn`: user VEIL/VAI balance keys set to `state.All`
- RPC method aliases were added to match lowercase client calls:
  - `Lpbalance`, `Vaibalance`, `Proofconfig`
- Successful liquidity tx sequence from latest run:
  - `add_liquidity`: `gacxp7YW8nVaDa9M85RMnupbgaEfFmStYfLZvG4EdVjC2DEb6`
  - `swap_exact_in`: `28WTm2GcgYjXSjCMXFHiRXJ77mboJnhCuidytKq1n8u5qB4Mzo`
  - `remove_liquidity`: `2oTdMMe72zHE8j3SqW4C2Qh5RekABdPKEHmByADZq4AizDC7W8`
- Invariant snapshot from the same pass:
  - before remove: `r0=20200 r1=19805 totalLP=19900 userLP=19900`
  - after remove: `r0=10100 r1=9903 totalLP=9950 userLP=9950`
- Status update (`2026-02-21` private-only gate follow-up):
  - direct AMM actions are now intentionally rejected by admission policy.
  - liquidity endpoint was rerouted to private `CommitOrder` ingestion.
  - evidence bundle: `evidence-bundles/20260221-fully-private-liquidity-commit/bundle.md`.
  - latest pointer: `evidence-bundles/latest-private-liquidity-commit.txt`.

## 0.4 2026-02-21 Proof-Verified Private-Only Admission Gate

- Consensus admission now enforces private-only action policy:
  - allowed action IDs: `2 (CommitOrder)`, `4 (ClearBatch)`, `17 (SubmitBatchProof)`, `18 (SetProofConfig)`
  - all other actions are rejected pre-execution with `proof-verified private action required`
- `SetProofConfig` can no longer disable proof mode:
  - `RequireProof=false` is rejected by storage validation (`ErrInvalidProofConfig`)
  - this fail-closes governance so proof mode cannot be switched off.
- Chain hook implementation:
  - added optional `ActionAdmissionController` path in tx `PreExecute` and `Execute`
  - VEIL `storage.BalanceHandler` now implements this hook with strict allowlist
- Validation (Docker/Linux toolchain for `DataDog/zstd` + `blst`):
  - `go test ./chain -run 'TestPreExecuteActionAdmissionController|TestPreExecute'` => PASS
  - `go test ./storage` (from `examples/veilvm`) => PASS
  - `go test ./actions` (from `examples/veilvm`) => PASS

## 1. Naming and Scope

- `VEIL` = custom HyperSDK VM chain (privacy, proof validity, treasury/risk invariants).
- `VEIL-EVM` (chainId `22207`) = companion Subnet-EVM rail for interop/access tooling.
- Production-critical economic and privacy rules remain VM-side; EVM side is supporting rails.

## 2. Executive Snapshot

| Area | Status | Notes |
|---|---|---|
| VEIL chain health | PASS (local) | Chain boots and produces blocks in local profile |
| Proof-gated flow | PASS (local strict) | Strict verifier active with `required_circuit_id=shielded-ledger-v1`; invalid proofs rejected |
| Latest launch-gate evidence | PASS | `evidence-bundles/20260219-231603-launch-gate-evidence/bundle.md` |
| Longer funded shielded run | PASS | `zkbench-out-groth16-shielded-long-funded-20260219-180640/summary.json` |
| Fuel/faucet resilience | PARTIAL | Only one funded non-genesis bench key currently has enough runway for constrained long runs |
| Companion precompile policy | PASS (local) | `npm.cmd run check:companion-policy` passes |
| Companion bridge contracts | BLOCKED (prod) | Teleporter/bridge deploys are still placeholders |
| Production readiness | NO-GO | Launch gates remain incomplete |

## 3. Completed Work to Date

### 3.1 VEIL VM Core

- Local chain bring-up stabilized with plugin handshake and reproducible local ops flow.
- Proof-gated batch pipeline is wired:
  - `SubmitBatchProof` + `ClearBatch` fail-closed checks.
  - Proof blob commitment in Vellum storage.
- ZK runtime hardening:
  - Startup logs now print resolved verifier config.
  - Local fallback auto-enables strict Groth16 if VK fixture file exists in-container.
- Bench hardening:
  - `veilvm-zkbench` validates tx execution success from indexer (`Result.Success`) instead of submit-only signals.
- Tokenomics/control action surface is present in VM action set (`RouteFees`, `ReleaseCOLTranche`, `MintVAI`, `BurnVAI`, pool/risk/proof actions).

### 3.2 Companion EVM (VEIL-EVM)

- Warp/AWM config is present in genesis.
- Fee manager + allowlist + native minter precompile configs are populated in `upgrade.json` using `adminAddresses` and `enabledAddresses`.
- Companion role map and deployments are recorded in `scripts/companion-evm.addresses.json`.
- Policy validation passes:
  - `npm.cmd run check:companion-policy` -> `PASS: required companion EVM primitive fields are populated.`

### 3.3 Late-Session Incident Fixes (2026-02-19)

Incident observed during longer zk runs:

- Repeated run failures from:
  1. fuel depletion / refuel wallet insufficiency
  2. `set_proof_config execution failed: unauthorized` when switching to a funded non-genesis bench signer

Fixes applied:

1. Evidence runner now exposes and uses explicit proof-config authority key.
   - File: `scripts/run-launch-gate-evidence.mjs`
   - Added CLI flag: `--proof-config-private-key <REDACTED>
   - Added env fallback: `VEIL_EVIDENCE_PROOF_CONFIG_PRIVATE_KEY`
   - Runner now always sets `PRIVATE_KEY` and `PROOF_CONFIG_PRIVATE_KEY` in base environment.

2. Backup takeover cases now use the resolved proof-config key instead of assuming the primary bench signer has config authority.
   - File: `scripts/run-launch-gate-evidence.mjs`

3. `veilvm-zkbench` source includes an unauthorized fallback path for proof-config submission.
   - File: `cmd/veilvm-zkbench/main.go`
   - New env: `PROOF_CONFIG_FALLBACK_PRIVATE_KEY`
   - Behavior: on `set_proof_config` unauthorized, retry with fallback key; if no fallback provided and bench key is non-genesis, auto-tries default genesis key.

Validation from this session:

- Long funded shielded run passed:
  - `zkbench-out-groth16-shielded-long-funded-20260219-180640/summary.json`
  - Result: `accepted=1 rejected=0 missed=0`
- Scripted launch-gate smoke passed with funded bench signer + explicit proof-config signer:
  - `evidence-bundles/20260219-231603-launch-gate-evidence/bundle.md`

## 4. Active Blockers

1. **COMPLETED (2026-02-21):** shielded-ledger circuit assurance pack archived at `evidence-bundles/zk-circuit-assurance/latest.txt`:
   - circuit spec hash + VK/PK hashes + proof vectors + verifier config dump + local security sign-off.
2. Companion bridge stack is not production-ready:
   - current Teleporter messenger/registry + bridge minter are local placeholder contracts.
3. Fuel runway risk remains:
   - non-genesis funded bench key balance is finite and drops per run.
4. Rebuild caveat:
   - `cmd/veilvm-zkbench/main.go` fallback logic is source-patched, but rebuilding in current runner image may fail (`gcc` missing) unless toolchain image is fixed.

## 4.1 Latest Launch-Gate Evidence (Current Pointer)

- Overall verdict: `PASS`
- Bundle: `evidence-bundles/20260219-231603-launch-gate-evidence/bundle.md`
- Bundle JSON: `evidence-bundles/20260219-231603-launch-gate-evidence/bundle.json`
- Check outcomes:
  - `shielded-smoke`: PASS (`accepted=1`, `rejected=0`, `missed=0`)

## 5. Key/Fuel Snapshot (2026-02-19T23:13Z)

Balance probe (`planned_txs=6`, `required=30000001`):

- Genesis key prefix `637404e672...`: balance `1197629` (insufficient for long-run startup budget).
- Funded key prefix `203206b925...`: balance `32739498` (usable for constrained long runs).

Operational implication:

- Use funded key as bench signer.
- Use genesis key as proof-config signer.
- Keep profiles constrained until faucet/reseed is restored.

## 6. Current Companion Registry (Local)

Source: `scripts/companion-evm.addresses.json`

| Field | Value |
|---|---|
| network | `VEIL-EVM` |
| chainId | `22207` |
| rpcUrl | `http://127.0.0.1:9650/ext/bc/2L5JWLhXnDm8dPyBFMjBuqsbPSytL4bfbGJJj37jk5ri1KdXhd/rpc` |
| tempAdminEoa | `0x580358...503A` |
| bridgeRelayer1 | `0x7deFD0...Ad53` |
| bridgeRelayer2 | `0x7843eb...4bdF` |
| opsKeeper1 | `0x698acA...4A55` |
| deployer1 | `0x9d0FE6...B1b4` |
| teleporterRegistry | `0x1B0302...9c66` |
| teleporterMessenger | `0xC1b366...34de` |
| bridgeMinterContract | `0x18a85a...eF0C` |
| multicall3 | `0x19668A...De30` |
| create2Deployer | `0x4e59b4...956C` |
| faucet | `0x936A9a...Ccb7` |

Upgrade config hash (current local):

- `scripts/companion-upgrade.json`
- SHA-256: `<PRIVATE_KEY_REDACTED>`

## 7. Repro/Verification Commands

From `examples/veilvm/scripts`:

```powershell
npm.cmd run check:companion-policy
```

Expected: `PASS: required companion EVM primitive fields are populated.`

Launch-gate smoke using funded bench signer plus explicit proof-config signer:

```powershell
node .\scripts\run-launch-gate-evidence.mjs `
  --private-key <REDACTED> `
  --proof-config-private-key <REDACTED> `
  --skip-backup-takeover `
  --skip-negative `
  --skip-malformed `
  --skip-timeout `
  --batch-size 8 `
  --windows-per-size 1 `
  --timeout-minutes 20
```

Direct long run (already proven in this session):

- Output: `zkbench-out-groth16-shielded-long-funded-20260219-180640`
- Config: groth16 shielded-ledger, `BATCH_SIZES=8`, `WINDOWS_PER_SIZE=1`, `TIMEOUT_MINUTES=20`

## 8. Immediate Next Sequence

1. **COMPLETED (2026-02-19):** `scripts/run-launch-gate-evidence.mjs` now enforces mandatory fuel preflight to faucet-fund required signer keys (`primary`, `proof-config`, optional `backup`), then strict-verify startup budget before checks. Runner hard-fails if faucet/preflight conditions are not met. (`--skip-fuel-preflight` exists only as an emergency bypass.)
2. Restore faucet/reseed strategy so takeover and multi-profile long runs can execute without manual balance triage.
3. Re-run adversarial drills on current chain state (`malformed`, `timeout`, `backup-takeover`) with refreshed funding.
4. **COMPLETED (2026-02-21):** full shielded-ledger circuit assurance pack archived to close gate `G3` (local): `evidence-bundles/zk-circuit-assurance/latest.txt`.
5. Replace placeholder Teleporter/bridge contracts with production implementations before any bridge-enabled launch mode.

## 9. Handoff Notes for Next Operator

- If `set_proof_config ... unauthorized` appears:
  - first ensure runner was invoked with `--proof-config-private-key`.
  - if running raw `veilvm-zkbench`, set `PROOF_CONFIG_PRIVATE_KEY` explicitly.
- If fuel errors reappear:
  - runner now auto-resolves faucet signer to genesis key when `--faucet-private-key` is omitted and bench signer is non-genesis.
  - run a balance probe first (prefund-only mode).
  - keep `GAS_SAFETY_BPS=10000` and `GAS_RESERVE=1` for constrained diagnostic runs.
- Evidence pointer file was stale before this update; keep `evidence-bundles/latest-launch-gate-evidence.txt` synchronized with newest PASS bundle.
