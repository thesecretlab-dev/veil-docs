# Phase A status (2026-08-24)

Operator: this PC (`C:\Users\Justin\src\veil`).  
Runlist: `VEIL_MAINNET_LAUNCH_RUNLIST.md`.

| ID | Status | Notes |
|---|---|---|
| A01 | **PASS** | Inventory: `veilvm/evidence-bundles/source-inventory/latest.md`. 42-action Josh tree **not** on disk. |
| A02 | **PASS** | Tag `veilvm-canonical-2026-08-24` → `b11da50`. |
| A03 | **PASS** (docs) | `specs/VEIL_ACTION_REGISTRY.md`. Binary = IDs 0–18. 19–41 spec-only. Handshake “42” is a claim, not this SHA. |
| A04 | **PASS** | `specs/VEIL_SUPPLY_FREEZE_2026-08-24.md` = **990,999,000**. Docs copies aligned. |
| A05 | **PASS** | `veilvm/evidence-bundles/STALE.md` + `latest-launch-gate-evidence.txt` marked `STALE-2026-02`. |
| A06 | **PASS** | `veilvm/evidence-bundles/key-map/latest.json`. Lost owner `0xB9a05AFC…96af` = do-not-deploy-under. No private keys on this PC. |
| A07 | **PASS** | Clones + `go build ./...` + `forge build` (34 contracts). Docker/WSL still missing (admin + reboot). |

**A exit** (A02, A03, A04, A06, A07): **PASS** except Docker for Phase C compose.

## Phase B (2026-08-24)

| ID | Status | Notes |
|---|---|---|
| B01 | **PARTIAL** | Go 1.26.7, gcc 16.1 MinGW, Node v24.19.0, Foundry 1.7.1. Docker Desktop / WSL not installed. |
| B02 | **FAIL** | `platform-cli` does not compile on Windows (`storage.AvailableBytes`). `avalanche-cli` source present, not built. |
| B03 | **PASS** | New operator keys generated 2026-08-24. Public map: `veilvm/evidence-bundles/key-map/operator-2026-08-24.json`. Private hex in `C:\Users\Justin\tools\veil-keybox\private` (not git). |
| B04 | TODO | Fund Fuji P/C for `pchain-operator` / `cchain-gas`. |
| B05 | **PASS** | Keybox outside git + ACL; `secrets/README.md` points at it. |

**Stack tighten (2026-08-24):** [`architecture/VEIL_STACK.md`](../architecture/VEIL_STACK.md). v1 = 19 actions. Companion = rails only. Parked Solidity is not the protocol.

## Phase C (2026-08-24, Windows local — no Docker)

Skipped Docker. Built AvalancheGo + VeilVM plugin natively.

| ID | Status | Notes |
|---|---|---|
| C01 | **PASS** (local Windows) | Node `:9660` healthy. Chain `bdRGUMA7rzZFXjbn1ePTjqhAUfTjW94e69p7qZd4puZ3uEosL` producing blocks. |
| C02 | **PASS** | `enabled=true strict=true groth16_vk_set=true required_circuit_id=shielded-ledger-v1` via chain config (plugin does not inherit `VEIL_ZK_*` env). |
| C03 | **PASS** (local Windows) | Native AMM smoke: mint VAI, add liquidity, swap. Order+liquidity dual-chain E2E PASS. |
| C04 | **PASS** (local anvil, not dual AvalancheGo EVM) | Companion = anvil `:8545` chainId 31337. Relayer + order-router in-tree. |
| C05 | **PASS** | `evidence-bundles/abandoned-feb-2026/`. Live `companion-evm.addresses.json` is anvil 31337. `check-companion-primitives` rejects chainId 22207. |
| C06 | **PASS** (local profile only) | `npm run check:prelaunch` `overallPass=true` `productionLaunchPass=false`. Artifact `control-tower/prelaunch-readiness-20260824184020.json`. |

Evidence: `veilvm/evidence-bundles/local-revive-2026-08-24/smoke.md`.

**C exit (local Windows):** C01, C02, C03, C06 PASS. DC2 (dual AvalancheGo EVM) tracked into Phase D/F.

## Phase D (2026-08-24)

| ID | Status | Notes |
|---|---|---|
| D01 | **PASS** (bucket sum) | circulating 49,549,950 + COL locked 900,000,000 + COL live 41,449,050 = 990,999,000. **Keep3r dropped** (0 bips, not a bucket). Artifact `evidence-bundles/launchpad-freeze/latest.txt`. |
| D02 | **PASS** | `go test ./actions -run ReleaseCOL` — unauthorized / zero / drain-all / epoch-cap / too-early all fail closed. Locked unchanged after drain attempt. |
| D03 | **PASS** | RouteFees 70/20/10 (`7000/2000/1000`) + remainder-to-ops. `PutFeeRouterConfig` rejects non-10000 bips. Genesis JSON freeze test. |
| D04 | **PASS** | VAI debt ceiling, epoch mint throttle + reset, backing floor, unauthorized mint. wsVEIL LTV must be 0 (`PutRiskConfig` / `SetRiskParams`). |
| D05 | **PASS** | `shielded-ledger-v1` Groth16 VK sha256 `40d25f181550c879f93d22dfa50305700bdb0e731ced46d1b789248e552398ba`. Sample proof verifies against pinned VK. `clearhash-v1` rejected when required circuit is shielded. Artifact `evidence-bundles/zk-circuit-assurance/latest.txt`. |
| D06–D10 | TODO | Privacy gossip, privacy-scope matrix, frontend verbiage, companion primitives, opaque-intent grep. |

Next: D06 encrypted gossip + threshold decrypt (shared-key-only = FAIL). Operator-only: B04 Fuji faucet (CAPTCHA). WSL/Docker still required for `platform-cli` and Fuji L1 (Phase E). Do not fund mainnet yet.
