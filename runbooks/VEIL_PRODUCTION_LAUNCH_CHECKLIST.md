# VEIL Production Launch Checklist

Status: Active  
Date: 2026-02-22  
Decision Mode: Hard Go/No-Go

## 1. Purpose

Define the exact production launch gates for:

- `VEIL` custom VM chain (privacy and consensus core)
- companion EVM chain (interop rails only)

Launch is blocked unless every gate below is `PASS` with archived evidence.

## 2. Owner Map

| Role | Responsibility |
|---|---|
| Protocol Lead | Final launch decision, parameter freeze, governance sign-off |
| VM Lead | VeilVM consensus rules, ZK verification path, invariant enforcement |
| Bridge/EVM Lead | AWM/Teleporter/bridge contracts, companion precompile policy |
| Treasury/Risk Lead | COL locks, VAI risk limits, fee router and RBS controls |
| SRE/Infra Lead | validator ops, uptime, monitoring, incident playbooks |
| Security Lead | audit closure, key management, adversarial testing sign-off |

## 3. Gate Board

| Gate ID | Gate | Primary Owner | PASS Criteria | FAIL Criteria | Required Evidence |
|---|---|---|---|---|---|
| G0 | Chain Health Baseline | SRE/Infra Lead | VEIL chain healthy, stable block production, deterministic restart success | unhealthy chain, re-bootstrap drift, unstable block acceptance | health snapshots, restart logs, chain IDs, config hashes |
| G1 | Proof-Gated Consensus | VM Lead | no-proof/no-clear enforced, invalid proof deterministically rejected, strict verifier path enabled | any market clear accepted without valid proof, nondeterministic verifier behavior | `zkbench` reports + rejection tests + verifier config dump |
| G2 | Native Privacy Invariants | VM Lead | encrypted batch flow + commitment/nullifier checks + fail-closed rules pass, and mempool gossip confidentiality uses threshold/network decryption (no single validator unilateral decrypt) | bypass path, stale/degraded privacy mode, shared-key-only mempool privacy in production, ambiguous acceptance rules | invariant tests + replay transcripts + `VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md` + threshold rollout audit bundle (`threshold-keying-rollout`) |
| G3 | Full ZK Circuit Scope | VM Lead + Security Lead | launch circuit includes full shielded-ledger constraints (not clear-hash benchmark-only path) and verifier circuit gate is set to `shielded-ledger-v1` | benchmark/demo circuit remains production path or circuit gate is unset/clearhash | circuit spec hash, VK hash, proof vectors, verifier config dump |
| G4 | Tokenomics + Treasury Locks | Treasury/Risk Lead | locked COL enforced at genesis, release caps enforced on-chain, Keep3r pool fixed at `2.0%` genesis allocation, no hidden mint/unlock route | treasury drain route, bypassable release caps, unfixed Keep3r allocation, supply mismatch | genesis hash, supply proof, treasury state proof, unlock simulation |
| G5 | VAI Risk Controls | Treasury/Risk Lead | debt ceiling, per-epoch mint throttle, backing floor, collateral haircuts all enforced | unbounded minting, broken peg controls, stale-oracle bypass | risk parameter snapshot + scenario test outputs |
| G6 | Companion EVM Bridge Readiness | Bridge/EVM Lead | real (non-placeholder) Teleporter/bridge contracts deployed, VEIL<->EVM round-trip tests pass, and EVM intent relay flows pass for both orders and liquidity (`IntentSubmitted -> /evm/intents/execute -> markIntentExecuted`, `LiquidityIntentSubmitted -> /evm/liquidity/execute -> markIntentExecuted`) | placeholder contracts, partial bridge path, replay/custody gaps, missing/failed intent relay flow | `scripts/companion-evm.addresses.json`, tx hashes, round-trip test logs, intent relay run logs |
| G7 | Companion Policy Hardening | Bridge/EVM Lead | `TxAllowList`, `ContractDeployerAllowList`, `NativeMinter` policy checks pass with correct role separation | policy mismatch, overprivileged roles, config hash drift | `npm run check:companion-primitives` + `npm run check:companion-policy` outputs |
| G8 | Security Audit Closure | Security Lead | critical/high findings resolved or accepted with explicit risk sign-off | unresolved critical/high issues without approved exception | audit reports, remediation diffs, exception register |
| G9 | Reliability + Failure Drills | SRE/Infra Lead | missed-proof, malformed-proof, prover-timeout, backup-takeover drills passed and documented | any drill untested or fails without mitigation | drill reports + incident playbook run logs |
| G10 | Key Ceremony + Admin Rotation | Security Lead + Protocol Lead | temporary EOAs rotated to multisig/HSM policy, final signer sets frozen | launch with temporary admin keys or undocumented signer ownership | signer manifest, key-ceremony record, final permission diff |
| G11 | Launch Rehearsal | Protocol Lead | full dry-run from genesis to bridge and market flow completed with deterministic outputs | partial rehearsal, inconsistent outputs, missing rollback plan | rehearsal report, rollback runbook, signed launch packet |
| G12 | ANIMA Runtime Readiness | ANIMA SDK Owner + Protocol Lead | ANIMA readiness bundle passes: encrypted wallet at rest, production-safe logging defaults, deterministic relay/runtime behavior checks, no simulated infra/validator milestone completion (fail-closed if integrations are missing), and launch-copy parity with no overclaims | plaintext key path active, unsafe payload/raw-stream logging enabled without explicit override, simulated milestone bypasses, failing runtime checks, or frontend overclaims | `evidence-bundles/anima-readiness/latest.txt` + `anima-readiness.json` + `anima-readiness.md` + supporting telemetry/signature artifacts |

## 4. Hard Launch Rule

All gates `G0` through `G12` are mandatory for production launch.

- If any gate is `FAIL` or `UNSET`: **NO-GO**
- If all gates are `PASS` with evidence: **GO**

## 5. Current Readiness Snapshot (2026-02-22)

| Gate ID | Status | Notes |
|---|---|---|
| G0 | PASS (local) | VEIL chain healthy and running in current local profile |
| G1 | PASS (local) | strict verifier is active; latest private-only smoke pass is archived at `evidence-bundles/20260222-033228-launch-gate-evidence/bundle.md` |
| G2 | PASS (local) | threshold-gated encrypted gossip + fail-closed keying are active in the local profile; latest rollout audit passes (`evidence-bundles/threshold-keying-rollout/latest.txt` -> `tkroll-20260222-190103`) with key-ceremony artifacts present (`evidence-bundles/key-ceremony/latest.txt`) |
| G3 | PASS (local) | shielded-ledger circuit assurance bundle is archived at `evidence-bundles/zk-circuit-assurance/latest.txt` with circuit spec hash, VK/PK hashes, proof vectors, verifier config dump, and local security sign-off |
| G4 | PASS (local) | launch tokenomics/treasury invariants pass prelaunch checks (2% Keep3r allocation, reserve-floor and supply consistency) and economic coherence latest is pass (`evidence-bundles/economic-coherence/latest.txt` -> `econ-20260222-140427`) |
| G5 | PASS (local) | VAI risk controls pass local gate checks: debt ceiling, per-epoch mint throttle, and backing floor invariants all pass in prelaunch + economic coherence evidence (`evidence-bundles/economic-coherence/latest.txt` -> `econ-20260222-140427`) |
| G6 | PASS (local) | companion opaque relay flows now pass for both orders and liquidity from seeded frontier replay range (`scripts/relay-opaque-intents.mjs --from-block 94`) with mark-executed completion; mainnet Teleporter/Chainlink live check is green (`scripts/check-live-mainnet-bridge.mjs`); failure/resolution trail archived at `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.md` |
| G7 | PASS (local) | `check:companion-primitives` and `check:companion-policy` both pass against local companion registry + `scripts/companion-upgrade.json` |
| G8 | PASS (local) | latest evmbench audit run `evmbench-20260220-141106` succeeded with 0 findings; flywheel audit run `flywheel-20260220-145840` now reports DAI/VAI suite coverage 10/10 with no critical/high findings |
| G9 | PASS (local) | private-only adversarial pass is archived at `evidence-bundles/20260222-024215-launch-gate-evidence/bundle.md` (`backup-takeover`, `malformed-proof`); `synthetic-negative` and `timeout-drill` also pass in `evidence-bundles/20260222-031447-launch-gate-evidence/bundle.md` |
| G10 | PASS | key ceremony and admin rotation pass (`evidence-bundles/key-ceremony/ceremony-20260221-184909/ceremony-manifest.json`, `evidence-bundles/admin-rotation/rotation-20260222-045726/ownership-rotation.json`); hardened owner 0xB9a05A...96af differs from temporary admin |
| G11 | PASS | launch rehearsal packet passes (`evidence-bundles/launch-rehearsal/rehearsal-20260222-095753/rehearsal-report.json`, `evidence-bundles/launch-rehearsal/rehearsal-20260222-095753/launch-packet.json`) with signature verification complete |
| G12 | PASS (local) | ANIMA readiness bundle now passes with strict-private fixture evidence mapped to Tier0 action IDs `2,3,17,4` on current local chain (`evidence-bundles/anima-readiness/latest.txt` -> `anima-20260222-143834`); supporting logs and fixture mapping are archived under `evidence-bundles/anima-readiness/anima-20260222-143834/` |

Current decision state: **GO FOR PRODUCTION**

## 6. Evidence Index (Required Paths)

- `VEIL_MASTER_RUNBOOK.md`
- `VEIL_EXECUTION_PACKAGE.md`
- `VEIL_V1_NATIVE_PRIVACY_SPEC.md`
- `VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md`
- `VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md`
- `VEIL_EVM_INTENT_RELAY_RUNBOOK.md`
- `EVMBENCH_AUDIT_RUNBOOK.md`
- `VEIL_ZK_CONSENSUS_4_6S_TRIAL_PROFILE.md`
- `VEIL_HANDOFF_2026-02-19.md`
- `evidence-bundles/20260221-mempool-gossip-hardening.md`
- `scripts/threshold-keying.rollout.template.json`
- `evidence-bundles/threshold-keying-rollout/latest.txt`
- `evidence-bundles/threshold-keying-rollout/tkroll-<timestamp>/threshold-keying-rollout.json`
- `evidence-bundles/threshold-keying-rollout/tkroll-<timestamp>/threshold-keying-rollout.md`
- `scripts/companion-evm.addresses.json`
- `evidence-bundles/latest-launch-gate-evidence.txt`
- `evidence-bundles/zk-circuit-assurance/latest.txt`
- `evidence-bundles/zk-circuit-assurance/zkassure-20260221-064537-shielded-ledger-v1/bundle.json`
- `evidence-bundles/zk-circuit-assurance/zkassure-20260221-064537-shielded-ledger-v1/bundle.md`
- `evidence-bundles/20260219-184441-launch-gate-evidence/bundle.json`
- `evidence-bundles/20260219-184441-launch-gate-evidence/bundle.md`
- `evidence-bundles/saved/20260219-184441-launch-gate-evidence.zip`
- `evidence-bundles/20260222-024215-launch-gate-evidence/bundle.json`
- `evidence-bundles/20260222-024215-launch-gate-evidence/bundle.md`
- `evidence-bundles/20260222-031447-launch-gate-evidence/bundle.json`
- `evidence-bundles/20260222-031447-launch-gate-evidence/bundle.md`
- `evidence-bundles/20260222-033228-launch-gate-evidence/bundle.json`
- `evidence-bundles/20260222-033228-launch-gate-evidence/bundle.md`
- `evidence-bundles/latest-g6-frontier-relay.md`
- `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.json`
- `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.md`
- `zkbench-out-groth16-live-1w/summary.json`
- `zkbench-out-groth16-live-32-64x3-deadline10s/summary.json`
- `evidence-bundles/audit-closure/latest.txt`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/job-status.json`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/audit-result.json`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/audit-closure-template.md`
- `evidence-bundles/launch-rehearsal/latest.txt`
- `evidence-bundles/launch-rehearsal/rehearsal-<timestamp>/rehearsal-report.json`
- `evidence-bundles/launch-rehearsal/rehearsal-<timestamp>/rehearsal-report.md`
- `evidence-bundles/launch-rehearsal/rehearsal-<timestamp>/rollback-runbook.md`
- `evidence-bundles/launch-rehearsal/rehearsal-<timestamp>/launch-packet.json`
- `evidence-bundles/launch-rehearsal/rehearsal-<timestamp>/launch-packet.md`
- `evidence-bundles/anima-readiness/latest.txt`
- `evidence-bundles/anima-readiness/anima-<timestamp>/anima-readiness.json`
- `evidence-bundles/anima-readiness/anima-<timestamp>/anima-readiness.md`
- `VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md`
- `scripts/build-dualchain-readiness-package.ps1`
- `scripts/dualchain-validator-stake.manifest.json`
- `scripts/dualchain-validator-stake.manifest.template.json`
- `evidence-bundles/dualchain-readiness/latest.txt`

## 7. Sign-Off Sheet

| Role | Name | Decision (`PASS`/`FAIL`) | Timestamp | Signature Ref |
|---|---|---|---|---|
| Protocol Lead | Josh | PASS | 2026-02-22T18:01:45-05:00 | evidence-bundles/mainnet-deployment/latest.txt |
| VM Lead | Josh | PASS | 2026-02-22T18:01:45-05:00 | evidence-bundles/critical-phase/critical-phase-20260222-224644/critical-phase-summary.json |
| Bridge/EVM Lead | Josh | PASS | 2026-02-22T18:01:45-05:00 | evidence-bundles/mainnet-deployment/bridge-live-check.json |
| Treasury/Risk Lead | Josh | PASS | 2026-02-22T18:01:45-05:00 | evidence-bundles/economic-coherence/econ-20260222-140427/economic-coherence.json |
| SRE/Infra Lead | Josh | PASS | 2026-02-22T18:01:45-05:00 | evidence-bundles/critical-phase/critical-phase-20260222-224644/critical-phase-summary.json |
| Security Lead | Josh | PASS | 2026-02-22T18:01:45-05:00 | evidence-bundles/key-ceremony/ceremony-20260221-184909/ceremony-manifest.json |
