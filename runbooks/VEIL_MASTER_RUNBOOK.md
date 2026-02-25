# VEIL Master Runbook (Execution Order)

Status: Active  
Date: 2026-02-19  
Owner: Protocol team

## 1. Goal

Ship VEIL as a privacy-native Avalanche L1 where:

- order flow is not visible before finalization
- balances are shielded with commitment/nullifier + ZK validity
- execution is fair (uniform batch auctions)
- tokenomics are anti-dump by design (low float, locked COL, deterministic unlocks)

## 2. Source Documents

Use these as the implementation stack in order:

1. `VEIL_V1_NATIVE_PRIVACY_SPEC.md` (protocol and consensus requirements)
2. `VEIL_EXECUTION_PACKAGE.md` (tokenomics, COL, risk controls, launch gates)
3. `VEIL_MASTER_RUNBOOK.md` (this execution order and ownership map)
4. `VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md` (encrypted gossip hardening status and threshold-decrypt next step)
5. `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md` (hard production go/no-go gates and sign-off board)
6. `VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md` (bridge/EVM primitive rollout gates)
7. `VEIL_EVM_INTENT_RELAY_RUNBOOK.md` (EVM user intent -> VEIL execution operations)
8. `VEIL_ZK_CONSENSUS_4_6S_TRIAL_PROFILE.md` (proof-required 4-6s latency trial override)
9. `VEIL_MAINNET_CHAINLINK_BRIDGE_RUNBOOK.md` (Avalanche mainnet + Chainlink bridge activation and live checks)
10. `VEIL_HANDOFF_2026-02-19.md` (project-wide current-state handoff)
11. `EVMBENCH_AUDIT_RUNBOOK.md` (contract audit harness workflow and G8 closure evidence)
12. `VEIL_FOUNDATION_FLYWHEEL_LAUNCH_PLAN.md` (foundation launch plan to accelerate liquidity and enforce flywheel KPIs in first 30 days)
13. `CRITICAL_PHASE_CONTROL_TOWER.md` (critical-phase control-plane sequence, artifact go/no-go criteria, and signature caveat)

## 3. Current Critical Blockers

## 3.1 Resolved Since Last Review

- Local VEIL VM runtime now boots cleanly in the local profile.
- RPCChainVM plugin handshake succeeds.
- VEIL chain reaches healthy block production.
- Local setup flow supports subnet-auth signing path and end-to-end chain creation.
- Local verifier runtime now logs effective ZK config at startup (`enabled/strict/vk/circuit`).
- Local strict fallback now auto-enables Groth16 verification when fixture VK is mounted in-container.
- `veilvm-zkbench` now validates tx execution success from indexer, preventing false-positive bench passes.
- `G3` shielded-ledger circuit assurance package is archived at `evidence-bundles/zk-circuit-assurance/latest.txt`.
- Encrypted mempool gossip transport is implemented with fail-closed key requirements and local secret-key: <REDACTED> (`VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md`, `evidence-bundles/20260221-mempool-gossip-hardening.md`).
- Private-only proof-path execution now supports marketless commit-order flow end-to-end:
  - `RevealBatch`, `SubmitBatchProof`, and `ClearBatch` no longer fail on missing market record (active status still enforced when market exists)
  - targeted adversarial gate pass archived at `evidence-bundles/20260222-024215-launch-gate-evidence/bundle.md`
  - smoke pass after private-only reveal-threshold override wiring fix archived at `evidence-bundles/20260222-033228-launch-gate-evidence/bundle.md`

## 3.2 Active Blockers

- 4-6s profile needs sustained benchmark evidence (high-window and long-run reports) after local launch-gate PASS.
- Consolidated single-bundle rerun of full private-only gate set (`smoke,backup,negative,malformed,timeout`) after reveal-threshold override wiring fix still needs archival.
- Companion EVM bridge path still uses placeholder Teleporter/bridge contracts and is not production-ready.
- Threshold-gated mempool decrypt release and cryptographic threshold-keying code path are implemented; key-ceremony tooling (`cmd/veilvm-keygen`) and rollout-audit automation (`scripts/audit-threshold-keying-rollout.mjs`) are available, and local two-validator rollout evidence now passes (`evidence-bundles/threshold-keying-rollout/latest.txt` -> `tkroll-20260222-190103`), but final production-validator-set ceremony + adversarial replay evidence remain pending before full-privacy launch claim.

## 4. Execution Plan

## Phase 0: Chain Bring-Up (Must pass before feature work)

Status: complete for local VEIL bring-up baseline.

1. Freeze compatible versions for:
   - AvalancheGo image
   - HyperSDK dependency set
   - VeilVM build target
2. Fix container/runtime ABI mismatch for plugin loading.
3. Validate local setup flow end-to-end on wiped data.
4. Record exact reproducible command sequence in docs.

Deliverable: deterministic local subnet bring-up with VeilVM plugin running.

## Phase 1: Protocol-Critical v1 Invariants

1. Enforce encrypted-mempool only path in production mode.
   - Current milestone: encrypted gossip transport + fail-closed key requirement is complete.
   - Remaining milestone: production activation of cryptographic threshold keying with signer ceremony and adversarial replay evidence.
   - Automation command for activation evidence: `cd scripts && npm run audit:threshold-keying -- --manifest ../evidence-bundles/key-ceremony/ceremony-<timestamp>/ceremony-manifest.json --rollout ./threshold-keying.rollout.json`
2. Enforce fail-closed batch execution:
   - no decrypt quorum -> reject batch
   - invalid proof -> reject batch
3. Enforce commitment/nullifier checks and solvency checks.
4. Enforce objective slashing paths only.
5. Enforce deterministic replay artifact outputs.

### Phase 1 Default: Proof-Gated Consensus Mode (v1)

Adopt this baseline unless governance approves a parameter override before launch:

- `batch_window = 5s`
- `proof_deadline = 10s` from batch close
- deterministic prover committee: `5 of 20` with `2` backup provers
- validity rule: no valid proof by deadline -> reject market batch (fail-closed)
- liveness rule: subnet MAY continue non-market/control blocks while market batch is rejected
- objective penalties:
  - missed proof deadline: slash `0.5%`
  - invalid proof submission: slash `5%`
  - repeated misses: jail per slashing policy

For the current low-latency proving trial, use `VEIL_ZK_CONSENSUS_4_6S_TRIAL_PROFILE.md` as the active override profile.

Design intent:

- keep privacy and proof verification inside consensus validity rules
- avoid requiring all 20 validators to generate full proofs every batch
- preserve predictable latency while keeping native privacy guarantees

Deliverable: v1 privacy and safety invariants implemented and test-covered.

## Phase 2: Tokenomics + COL at Genesis

1. Finalize exact genesis allocation values (no placeholders).
2. Encode locked COL vault in VM state at genesis.
3. Allow release only via `ReleaseCOLTranche` with epoch cap.
4. Route fees 70/20/10 on-chain with auditable accounting.
5. Implement Olympus-style RBS policy executor with hard risk gates.
6. Implement native UniV2-style pool primitives (VEIL/VAI first) at VM level.
7. Enforce exogenous-reserve-only backing floor for VAI minting.
8. Keep vVEIL collateral-disabled in v1 (`LTV = 0`) and gate VEIL/WVEIL with risk params.
9. Wire dashboard feeds for float, unlocks, COL NAV, and risk utilization.
10. Reserve `KEEP3R_PROGRAM_POOL = 2.0%` (`23,828,981 VEIL`) at genesis under foundation-controlled bounded policy.
11. Use `scripts/genesis-launchpad.mjs` as the threshold gate that freezes tokenomics and launch artifacts once AVAX commitments hit target.

Deliverable: anti-dump, low-float launch state enforced by consensus rules.

## Phase 2.5: Interop + EVM Companion Rails

This phase is required if VEIL ships contract rails in parallel to VeilVM.

1. Deploy Avalanche Warp Messaging path (`AWM`) and Teleporter contracts for cross-chain messaging.
2. Define canonical bridge routes for VEIL-family and VAI wrappers between VeilVM and companion EVM rails.
3. On companion EVM chain config, enable and test:
   - `NativeMinter`
   - `FeeConfigManager`
   - `ContractDeployerAllowList`
   - `TxAllowList`
4. Deploy base EVM utility contracts expected by tooling:
   - `WVEIL` wrapper
   - `Multicall3`
   - `CREATE2` deterministic deployer (`0x4e59...B4956C`)
   - faucet service for dev/test environments
5. Freeze chain config timestamps, fee recipients, and admin keys before public launch.
6. Complete every MUST test in `VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md` and archive evidence.
7. Populate `scripts/companion-evm.addresses.json` from template and commit addresses + tx hashes.
8. Run and archive the EVM intent relay path defined in `VEIL_EVM_INTENT_RELAY_RUNBOOK.md`.
9. Run and archive the economic flywheel runtime audit:
   - command: `cd scripts && npm run audit:flywheel`
   - outputs: `evidence-bundles/flywheel-audit/flywheel-*/flywheel-audit.{json,md}`
   - hard requirement: latest run must be pass before launch sign-off.
10. Enable Keep3r jobs for foundation infra targets (`VeilTreasury`, order intent gateway, liquidity intent gateway) and archive tx hashes in evidence.
11. Rotate EVM admin ownership away from temporary EOA before launch:
   - dry-run: `cd scripts && npm run admin:rotate -- --new-owner <MULTISIG_OR_HSM_EOA>`
   - execute: `cd scripts && npm run admin:rotate -- --new-owner <MULTISIG_OR_HSM_EOA> --execute`
   - outputs: `evidence-bundles/admin-rotation/rotation-*/ownership-rotation.{json,md}`
   - hard requirement: owner of `VAI`, `VeilTreasury`, and `VeilKeep3r` must equal hardened owner address.
12. Run and archive economic coherence audit:
   - command: `cd scripts && npm run audit:economics`
   - outputs: `evidence-bundles/economic-coherence/econ-*/economic-coherence.{json,md}`
   - hard requirement: supply/risk math invariants pass and reference-trade slippage target is met or explicitly waived with sign-off.
13. Run and archive VM/privacy mechanism audit:
   - command: `cd scripts && npm run audit:vm:privacy`
   - outputs: `evidence-bundles/vm-privacy-audit/vmprivacy-*/vm-privacy-audit.{json,md}`
   - hard requirement: shielded smoke, fail-close mismatch rejection, malformed-proof rejection, timeout enforcement, and backup takeover all pass.
14. If slippage target is not met, run deterministic liquidity deepening:
   - command: `cd scripts && npm run liquidity:deepen`
   - output: `evidence-bundles/liquidity-depth/liquidity-*/liquidity-depth.json`
   - rerun `audit:economics` after depth update.

Hard split of responsibility:

- VeilVM core enforces privacy, batch validity, COL locks, VAI risk gates, and treasury invariants.
- EVM precompiles and bridge contracts provide interoperability and permissioning rails.
- No critical economic invariant may depend only on companion EVM precompile settings.

## Phase 3: SLO and Launch Readiness

1. Run stress and adversarial simulation suite.
2. Run failure playbooks and governance emergency drills.
3. Verify telemetry and disclosure pipeline.
4. Perform launch gate sign-off.
5. Execute the 30-day launch cadence defined in `VEIL_FOUNDATION_FLYWHEEL_LAUNCH_PLAN.md` (liquidity ignition, KPI triggers, emissions tightening timeline).

Deliverable: production readiness package with objective pass/fail evidence.

### Critical-Phase Control-Plane Command Order

Run from `examples/veilvm/scripts`:

1. `npm run check:prelaunch`
2. `npm run launch:rehearsal`
3. `node run-critical-phase-gates.mjs` (optional consolidated summary runner; re-runs gates in isolated `evidence-bundles/critical-phase` output)

Command truth state (`2026-02-22`):

- `check:prelaunch` and `launch:rehearsal` are defined in `scripts/package.json`.
- `run-critical-phase-gates` exists as `scripts/run-critical-phase-gates.mjs` and is invoked as a direct `node` command.

Critical-phase no-go conditions:

- `check:prelaunch` does not return `overallPass=true`.
- `launch:rehearsal` output packet is unsigned-bypass (`allowUnsigned=true`) or fails signature policy (`signatureCheckPass=false`).
- Any required rehearsal check fails in `rehearsal-report.json`.

See `CRITICAL_PHASE_CONTROL_TOWER.md` for artifact-level criteria.

## 5. VM-Level COL Design (Genesis-First)

## 5.1 Required State Objects

- `COL_VAULT_LOCKED`
- `COL_VAULT_LIVE`
- `COL_RELEASE_SCHEDULE`
- `COL_RISK_LIMITS`
- `FEE_ROUTER_STATE`
- `RBS_POLICY_STATE`

## 5.2 Hard Rules

- No transfer from locked vault except `ReleaseCOLTranche`.
- `ReleaseCOLTranche` must enforce per-epoch max release.
- RBS execution must reject on stale data, cap breaches, or drawdown breach.
- Governance cannot bypass locked-vault no-drain rule.

## 5.3 Genesis Procedure

1. Compute and freeze allocation table.
2. Write allocations and lock flags directly in genesis state.
3. Set release schedule and caps in genesis parameters.
4. Set fee router defaults in genesis parameters.
5. Run launchpad derivation (`committedUSD -> reserve/debt/epoch limits`) and freeze output artifacts.
6. Verify invariant sum equals total supply.
7. Run genesis validation script and archive output hash.

## 6. Launch Interaction Model (VAI + Olympus)

At genesis, the economic core is two native VM modules:

- `VAIEngine` (Maker-style stable issuance)
- `VeilTreasury` (bonding, COL deployment, and RBS control)

They run as one coupled system with three loops:

1. Issuance loop (`VAIEngine`):
   - `MintVAI`, `BurnVAI`, `LiquidateCDP`
   - strict debt ceiling and per-epoch mint throttle
2. Treasury loop (`VeilTreasury`):
   - `BondDeposit`, `BondRedeem`, `DeployCOL`, `RebalanceCOL`, `BuybackMake`
   - vested claims only; no instant liquid VEIL emissions
3. Controller loop (`RiskKernel` + `FeeRouter`):
   - peg/stability controls, stale-oracle rejects, drawdown breakers
   - fee flow kept on-chain and auditable (`70/20/10`)

Launch sequence at T0:

1. Initialize `VAIEngine`, `VeilTreasury`, `RiskKernel`, and `FeeRouter` in genesis state.
2. Keep VEIL float constrained via locked treasury state and deterministic release caps.
3. Allow conservative VAI minting under debt/LTV bounds.
4. Route bond inflows to treasury reserves and deploy COL under strict risk caps.
5. Auto-pause mint/bond/RBS paths on peg breaks, stale oracle, or drawdown breaches.

Hard requirements:

- no direct drain path from locked treasury
- no instant bond payout path
- bounded governance parameter ranges with timelock
- non-reflexive launch preference (exogenous collateral support at genesis)

## 7. Tokenomics Integrity Controls

To preserve low float and runway:

- fixed total supply
- no hidden mint routes
- deterministic unlocks only
- bounded governance parameter ranges
- mandatory timelock on economic changes
- public real-time accounting outputs

### Approved Numeric Baseline (Current)

Use these exact constants unless governance formally revises them before genesis:

- `TOTAL_SUPPLY = 1,191,449,050 VEIL`
- `KEEP3R_PROGRAM_POOL = 2.0%` (`23,828,981 VEIL`)
- financing raise target: `$2,000,000`
- sold in seed+presale: `5.0%` (`49,549,950 VEIL`)
- blended financing price: `$0.04036331`
- seed: `2.0%` at `$0.03509853` (`$695,652.17`)
- presale: `3.0%` at `$0.04387316` (`$1,304,347.83`)
- raise usage split: `65%` stability reserve / `25%` COL-RBS / `10%` ops-security
- vVEIL policy bands: `18-24%` (M0-3), `14-18%` (M4-6), `10-14%` (M7-12), `8-12%` steady
- vVEIL hard controls: APY cap `30%`, emission cap `<= 4%` supply/year, unbond cooldown `14 days`

## 8. Testing Program (Minimum)

1. Runtime compatibility and chain boot tests
2. Setup script integration tests on fresh data
3. Batch privacy and proof failure-path tests
4. Nullifier/double-spend rejection tests
5. COL release-cap and no-drain invariant tests
6. RBS stress tests (volatility, stale-oracle, cap breach)
7. Replay determinism and selective-disclosure tests

## 9. Collaboration Rules (Shared Repo)

- Claim files before editing.
- Keep diffs narrow to owned scope.
- Avoid cross-file reformat churn.
- Treat genesis artifacts as coordinated edits.

## 10. Immediate Next Actions

1. Keep verifier mode pinned to `shielded-ledger-v1` in local profile and archive verifier config dumps with each evidence run.
2. Expand from one-window launch-gate PASS to sustained benchmark matrix coverage (32/64/96/128) and publish p50/p95/p99 artifact sets.
3. Re-run and archive a consolidated full private-only launch-gate bundle (`smoke,backup,negative,malformed,timeout`) after reveal-threshold override wiring fix.
4. `G3` circuit assurance artifacts are now archived at `evidence-bundles/zk-circuit-assurance/latest.txt`; refresh/re-sign this bundle after any circuit or key rotation.
5. Finalize numeric genesis allocation table for COL strategy and enforce invariants in tests.
6. Complete COLVaultModule + ReleaseCOLTranche hard invariants.
7. Implement RBSModule with strict risk-<REDACTED> execution.
8. Complete AWM/Teleporter + companion EVM precompile and allowlist rollout tests.
9. Finalize VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md with committed address registry and tx hash evidence.
10. Encode and test `KEEP3R_PROGRAM_POOL` genesis accounting and bounded payout controls.
11. Run `npm run audit:flywheel` and require a passing report in `evidence-bundles/flywheel-audit/latest.txt`.
12. Run `npm run audit:economics` and require a passing report in `evidence-bundles/economic-coherence/latest.txt`.
13. Run `npm run audit:vm:privacy` and require a passing report in `evidence-bundles/vm-privacy-audit/latest.txt`.
14. Use `npm run liquidity:deepen` when slippage KPI fails and archive resulting depth artifact.
15. Run `npm run check:prelaunch` then `npm run launch:rehearsal` (and optionally `node run-critical-phase-gates.mjs` for consolidated summary); require `rehearsal-report.json.overallPass=true`, `launch-packet.json.signatureCheckPass=true`, and `launch-packet.json.allowUnsigned=false` before any go decision.
