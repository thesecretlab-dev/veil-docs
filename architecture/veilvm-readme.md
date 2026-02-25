# VeilVM

Privacy-native prediction markets on Avalanche, built with HyperSDK.

## Overview

VeilVM implements the VEIL protocol - a custom Avalanche VM for privacy-preserving prediction markets using commit-reveal batch auctions.

## Key Launch Docs

- `VEIL_MASTER_RUNBOOK.md` (execution order and ownership map)
- `CRITICAL_PHASE_CONTROL_TOWER.md` (critical-phase control-plane command order + artifact go/no-go criteria)
- `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md` (go/no-go gates)
- `VEIL_STANDALONE_RPC_RUNBOOK.md` (operator path for exposing self-host VEIL RPC to cloud runners)
- `VEIL_CHILD_NODE_BOOTSTRAP_RUNBOOK.md` (repeatable dependency bootstrap for each child validator node)
- `VEIL_FOUNDATION_FLYWHEEL_LAUNCH_PLAN.md` (foundation launch plan for flywheel ignition, liquidity acceleration, and 30-day KPI controls)
- `EVMBENCH_AUDIT_RUNBOOK.md` (contract audit artifact process)
- `VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md` (encrypted gossip hardening status, validation, and threshold-decrypt next step)

## Current Status (2026-02-22)

- Local VEIL chain is running with coreapi and veilapi routes live.
- Proof-gated batch pipeline is active:
  - SubmitBatchProof is required for proof records.
  - ClearBatch enforces threshold reveal shares, commitment hash, and canonical public-input hash checks.
- Vellum proof blob storage is wired and queryable.
- ZK timing metrics are instrumented in actions and exposed over JSON-RPC.
- Consensus verifier wiring is active in the local profile:
  - Groth16/PLONK verifier path (gnark, BN254) is installed in-VM
  - runtime config is logged at startup for explicit verifier state
  - local fallback auto-enables strict Groth16 verification when fixture VK exists at `/root/.avalanchego/zk/groth16_clearhash_vk.bin`
  - SubmitBatchProof verifies before state writes when verifier is enabled
- Bench harness cmd/veilvm-zkbench is running and producing:
  - metrics_batch_32.csv, metrics_batch_64.csv, metrics_batch_96.csv, metrics_batch_128.csv
  - summary.md, summary.json
- Bench tx validation now checks indexed execution result (`Result.Success`) instead of only submit + height advance.
- Native liquidity ingress is now private-commit only on local runtime (`2026-02-21`):
  - `/intents/native/liquidity/execute` commits opaque liquidity envelopes as `CommitOrder` (no direct public `AddLiquidity/Swap` action path).
  - evidence: `evidence-bundles/20260221-fully-private-liquidity-commit/bundle.md`.
  - post-restart evidence: `evidence-bundles/20260221-132221-private-liquidity-post-restart/bundle.md`.
  - sender hardening: `CommitOrder` execution now requires treasury `Operations` signer; end-user addresses are not on-chain senders.
- Proof-verified private transaction gate is now enforced at consensus admission (`2026-02-21`):
  - accepted actions: `CommitOrder (2)`, `RevealBatch (3)`, `SubmitBatchProof (17)`, `ClearBatch (4)`, `SetProofConfig (18)`, `StakeVEIL (31)`, `SetRevealCommittee (41)`
  - all other action type IDs are rejected with `proof-verified private action required`
  - `SetProofConfig` can no longer disable proof mode (`RequireProof=false` is rejected)
- Encrypted tx gossip for mempool relay is now supported and can be hard-required (`2026-02-21`):
  - gossip payload format is authenticated-encrypted (`AES-256-GCM`) before p2p relay
  - when required and key is missing/invalid, node startup fails closed
  - local docker profile now enables `VEIL_TX_GOSSIP_ENCRYPTION_REQUIRED=true` and reads key from local `examples/veilvm/.env`
  - evidence: `evidence-bundles/20260221-mempool-gossip-hardening.md`
  - hardening runbook: `VEIL_MEMPOOL_PRIVACY_HARDENING_RUNBOOK.md`
- Private-only proof-path blocker is closed on current runtime (`2026-02-22`):
  - `RevealBatch`, `SubmitBatchProof`, and `ClearBatch` now support marketless private commit-order flow (still enforcing active market status when a market record exists).
  - local node image was rebuilt/restarted so chain execution uses patched VM logic.
- Latest targeted private-only evidence:
  - adversarial pass (`backup`, `malformed`): `evidence-bundles/20260222-024215-launch-gate-evidence/bundle.md`
  - smoke pass after reveal-threshold override wiring fix: `evidence-bundles/20260222-033228-launch-gate-evidence/bundle.md`
  - full-set pre-fix run (overall fail only on smoke threshold): `evidence-bundles/20260222-031447-launch-gate-evidence/bundle.md`

Not complete yet:

- Sustained multi-window/long-run 4-6s profile evidence is still pending.
- Consolidated single-bundle rerun for full private-only gate set (`smoke,backup,negative,malformed,timeout`) after the reveal-threshold Docker wiring fix is pending archive.
- Cryptographic threshold keying path is implemented; local two-validator ceremony + rollout audit now passes (`evidence-bundles/threshold-keying-rollout/latest.txt` -> `tkroll-20260222-190103`), while final production-validator-set ceremony + adversarial replay evidence remain pending before full-privacy launch claim.
- Production gate `G3` circuit package is archived at `evidence-bundles/zk-circuit-assurance/latest.txt` (`spec hash + VK/PK hashes + proof vectors + verifier config dump + local security sign-off`).

## Actions

| Action | ID | Description |
|--------|-----|-------------|
| Transfer | 0 | Transfer VEIL tokens |
| CreateMarket | 1 | Create a new prediction market |
| CommitOrder | 2 | Submit encrypted order commitment |
| RevealBatch | 3 | Submit decryption share for batch reveal |
| ClearBatch | 4 | Clear a batch auction (proof-gated) |
| ResolveMarket | 5 | Resolve market with oracle attestation |
| Dispute | 6 | Dispute a market resolution |
| RouteFees | 7 | Split protocol fees across MSRB/COL/Ops |
| ReleaseCOLTranche | 8 | Release treasury COL by epoch cap |
| MintVAI | 9 | Mint VAI against configured reserve/risk gates |
| BurnVAI | 10 | Burn VAI and repay caller debt position |
| CreatePool | 11 | Create UniV2-style pool |
| AddLiquidity | 12 | Add liquidity to pool |
| RemoveLiquidity | 13 | Remove liquidity from pool |
| SwapExactIn | 14 | Exact-in swap in pool |
| UpdateReserveState | 15 | Governance update of reserve telemetry |
| SetRiskParams | 16 | Governance update of collateral/risk params |
| SubmitBatchProof | 17 | Submit batch proof + Vellum proof blob |
| SetProofConfig | 18 | Governance update of proof requirements |
| BondDeposit | 19 | Bond VAI into VeilTreasury buffer |
| BondRedeem | 20 | Redeem bonded VAI from VeilTreasury buffer |
| CreateBondMarket | 21 | Create a typed bond market (Reserve/Inverse/VEIL/Liquidity) |
| PurchaseBond | 22 | Purchase a bond note from an active bond market |
| RedeemBondNote | 23 | Redeem matured bond note payout |
| SetYRFConfig | 24 | Configure Yield Repurchase Facility control params |
| RunYRFWeeklyReset | 25 | Run weekly YRF rollover |
| RunYRFDailyBeat | 26 | Run daily YRF buyback beat |
| SetRBSConfig | 27 | Configure Range-Bound Stability params |
| TickRBS | 28 | Process one RBS epoch observation |
| LiquidateCDP | 29 | Liquidate undercollateralized debt by repaying VAI for seized VEIL collateral |
| SetVVEILPolicy | 30 | Governance-configure dynamic vVEIL APY/rebase controller |
| StakeVEIL | 31 | Stake VEIL into rebasing vVEIL shares |
| WrapVVEIL | 32 | Wrap rebasing vVEIL shares into non-rebasing gVEIL |
| UnwrapGVEIL | 33 | Unwrap gVEIL back into vVEIL shares |
| RebaseVVEIL | 34 | Execute one policy-driven vVEIL index rebase epoch |
| SetVAIVaultConfig | 35 | Governance-configure permissionless VAI CDP stability params |
| DepositCDP | 36 | Lock VEIL collateral into caller CDP |
| WithdrawCDP | 37 | Unlock VEIL collateral from caller CDP (health-gated) |
| DrawVAIFromCDP | 38 | Permissionless VAI draw against caller CDP collateral |
| RepayVAIToCDP | 39 | Repay VAI debt on caller CDP |
| AccrueVAIStability | 40 | Advance VAI stability fee index and vault debt totals |
| SetRevealCommittee | 41 | Governance-set reveal committee member for validator index |

### Strict Private-Only Admission

- Action implementations still exist, but runtime admission is now private-only.
- Consensus currently admits only:
  - `CommitOrder` (`2`)
  - `RevealBatch` (`3`)
  - `SubmitBatchProof` (`17`)
  - `ClearBatch` (`4`)
  - `SetProofConfig` (`18`)
  - `StakeVEIL` (`31`)
  - `SetRevealCommittee` (`41`)
- Any other action ID is rejected before execution with `proof-verified private action required`.
- Live admission profile can be queried over JSON-RPC with `veilvm.privateadmission`.
- Scripted check: `cd scripts && npm run check:private-admission -- --chain-id <CHAIN_ID>`.

### Encrypted Mempool Gossip

- Configure tx gossip encryption via VM config or environment:
  - `txGossipEncryptionKeyHex` (64 hex chars / 32 bytes)
  - `txGossipEncryptionRequired` (`true|false`)
  - `txGossipThresholdDecryptEnabled` (`true|false`)
  - `txGossipThresholdMinShares` (integer, must be `>=2` when threshold mode is enabled)
  - `txGossipThresholdNodePrivateKeyHex` (local X25519 private key <REDACTED> cryptographic threshold mode)
  - `txGossipThresholdCommitteePublicKeys` (map of `NodeID` -> X25519 public key hex)
- Environment overrides (in order):
  - `VEIL_TX_GOSSIP_ENCRYPTION_KEY_HEX`, `VEIL_TX_GOSSIP_ENCRYPTION_REQUIRED`
  - `HYPERSDK_TX_GOSSIP_ENCRYPTION_KEY_HEX`, `HYPERSDK_TX_GOSSIP_ENCRYPTION_REQUIRED`
  - `VEIL_TX_GOSSIP_THRESHOLD_DECRYPT_ENABLED`, `VEIL_TX_GOSSIP_THRESHOLD_MIN_SHARES`
  - `HYPERSDK_TX_GOSSIP_THRESHOLD_DECRYPT_ENABLED`, `HYPERSDK_TX_GOSSIP_THRESHOLD_MIN_SHARES`
  - `VEIL_TX_GOSSIP_THRESHOLD_NODE_PRIVATE_KEY_HEX`, `VEIL_TX_GOSSIP_THRESHOLD_COMMITTEE_PUBLIC_KEYS_JSON`
  - `HYPERSDK_TX_GOSSIP_THRESHOLD_NODE_PRIVATE_KEY_HEX`, `HYPERSDK_TX_GOSSIP_THRESHOLD_COMMITTEE_PUBLIC_KEYS_JSON`
- If `txGossipEncryptionRequired=true` and no key is provided, VM init fails.
- If a key is provided, tx gossip payloads are sent as encrypted envelopes instead of plaintext batched tx bytes.
- If threshold mode is enabled, encrypted tx envelopes are only released into mempool submit path after quorum (`txGossipThresholdMinShares`) validator attestations are observed.
- If threshold node key + committee keys are configured, cryptographic threshold keying mode is used:
  - sender derives per-envelope data key
  - data key is Shamir-split and each share is X25519-encrypted to a committee validator
  - no single validator's local key material is sufficient before threshold shares are combined
- Local docker profile reads threshold settings from `examples/veilvm/.env` and materializes per-chain VM config files at startup (`docker-compose.local.yml`).
  - `VEIL_VM_CHAIN_IDS` controls which chain IDs receive generated `config.json` (defaults to active local VEIL chain IDs).
  - this is required because AvalancheGo plugin subprocesses do not inherit arbitrary container env vars.
- Current scope: encrypted transport + network quorum-gated decryption release + optional cryptographic threshold keying path.
- Remaining blocker for fully private mempool launch claim: production key ceremony + activation + adversarial evidence for cryptographic threshold keying path.

Threshold key ceremony tooling (`cmd/veilvm-keygen`):

```bash
# Generate one validator threshold node key pair (share public key with security lead)
go run ./examples/veilvm/cmd/veilvm-keygen --mode threshold-node --node-id <NodeID-...> --out ./examples/veilvm/evidence-bundles/key-ceremony/node-keys

# Assemble committee map + manifest (+ per-node env snippets when private key <REDACTED> is provided)
go run ./examples/veilvm/cmd/veilvm-keygen \
  --mode threshold-ceremony \
  --committee-node-ids <NodeID-A>,<NodeID-B>,<NodeID-C> \
  --min-shares 2 \
  --committee-public-keys-file ./examples/veilvm/evidence-bundles/key-ceremony/committee-public-input.json \
  --committee-private-keys-file ./examples/veilvm/evidence-bundles/key-ceremony/committee-private-input.json \
  --out ./examples/veilvm/evidence-bundles/key-ceremony/ceremony-<timestamp>
```

Production recommendation (Docker-first; avoids local `zstd`/`blst` CGO issues):

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm

powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run-production-key-ceremony.ps1 `
  -CommitteeNodeIDs "NodeID-A,NodeID-B,NodeID-C" `
  -PublicRecordsDir ".\evidence-bundles\key-ceremony\node-keys" `
  -MinShares 2 `
  -PromoteLatest
```

Per-validator runtime env build (run on validator host):

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\build-threshold-node-env.ps1 `
  -NodeID "NodeID-A" `
  -CeremonyDir ".\evidence-bundles\key-ceremony\latest" `
  -PrivateEnvPath ".\secure\NodeID-A.private.env" `
  -OutFile ".\secure\runtime\NodeID-A.env"
```

Full procedure:

- `examples/veilvm/VEIL_PRODUCTION_KEY_CEREMONY_RUNBOOK.md`

Threshold rollout activation audit automation:

```bash
cd ./examples/veilvm/scripts
cp threshold-keying.rollout.template.json threshold-keying.rollout.json
# fill in real validator node IDs, envFile paths, readiness URLs, and log sources
npm run audit:threshold-keying -- \
  --manifest ../evidence-bundles/key-ceremony/ceremony-<timestamp>/ceremony-manifest.json \
  --rollout ./threshold-keying.rollout.json \
  --log-tail 2500
```

Audit outputs:

- `evidence-bundles/threshold-keying-rollout/tkroll-*/threshold-keying-rollout.json`
- `evidence-bundles/threshold-keying-rollout/tkroll-*/threshold-keying-rollout.md`
- `evidence-bundles/threshold-keying-rollout/latest.txt`

Critical-phase control-plane operator sequence (`G11` path):

```bash
cd ./examples/veilvm/scripts
npm run check:prelaunch
npm run launch:rehearsal
# optional consolidated summary wrapper (runs prelaunch + rehearsal sequence in one report)
node run-critical-phase-gates.mjs
```

Command truth state (`2026-02-22`):

- `check:prelaunch` and `launch:rehearsal` exist in `scripts/package.json`.
- `run-critical-phase-gates` exists as `scripts/run-critical-phase-gates.mjs` (direct `node` command; no `npm run` alias).
- Signature bypass (`--allow-unsigned` / `VEIL_LAUNCH_PACKET_ALLOW_UNSIGNED=true`) is dry-run only and not launch-authorizing.

Control-plane go/no-go criteria and signature policy details:

- `CRITICAL_PHASE_CONTROL_TOWER.md`

Rehearsal outputs:

- `evidence-bundles/launch-rehearsal/rehearsal-*/rehearsal-report.json`
- `evidence-bundles/launch-rehearsal/rehearsal-*/rehearsal-report.md`
- `evidence-bundles/launch-rehearsal/rehearsal-*/rollback-runbook.md`
- `evidence-bundles/launch-rehearsal/rehearsal-*/launch-packet.json`
- `evidence-bundles/launch-rehearsal/rehearsal-*/launch-packet.md`
- `evidence-bundles/launch-rehearsal/latest.txt`

## Bond Markets (VEIL-Native)

Bond markets are typed and mapped to VEIL/VAI assets:

- `Reserve`: user pays `VAI`, receives `VEIL` at redemption.
- `Inverse`: user pays `VEIL`, receives `VAI` (instant-vesting note).
- `VEIL`: user pays `VEIL`, receives `VEIL` (time-locked staking form).
- `Liquidity`: user pays LP position units, receives `VEIL` at redemption.

Core action flow:

1. Governance creates a market with `CreateBondMarket`.
2. User buys a note with `PurchaseBond`.
3. User claims after maturity with `RedeemBondNote`.

Pricing model:

- Market price decays over time and steps upward after each purchase.
- `Reserve/VEIL/Liquidity` payout math: `payout = quote * 10_000 / priceBips`
- `Inverse` payout math: `payout = quote * priceBips / 10_000`

Treasury reservation model:

- Payout liquidity is reserved at purchase time.
- This removes redemption-time treasury drain races and fail-open behavior.

JSON-RPC helpers:

- `bondstate`: total outstanding bonded payout + current VAI buffer view.
- `bondmarket`: typed market config/state by `market_id`.
- `bondnote`: note ownership/vesting/claim status by `note_id`.
- `treasurylp`: treasury-held LP balance for a pair.

## Stability Loops

YRF:

- `SetYRFConfig` configures heart authority and spend controls.
- `RunYRFWeeklyReset` stages/activates weekly budget.
- `RunYRFDailyBeat` executes a daily spend + buyback/burn accounting beat.

RBS:

- `SetRBSConfig` configures MA window, spreads, capacities, and regeneration thresholds.
- `TickRBS` updates MA/bounds/capacities and toggles cushion market activation state.

Stability RPC helpers:

- `yrf`: current YRF config + state.
- `rbs`: current RBS config + state.
- `vveilstate`: current vVEIL policy, index, and last rebase outputs.
- `vveilbalance`: per-address vVEIL shares + derived rebasing amount.
- `gveilbalance`: per-address gVEIL units.
- `vaivault`: VAI vault config + global stability/debt share state.
- `cdpposition`: per-address CDP collateral/debt/health view.

Reference: see `BOND_MARKETS_V2.md` for implementation details and stability primitive semantics.

## Genesis Launchpad Trigger

Use `scripts/genesis-launchpad.mjs` to freeze launch parameters and gate launch on AVAX commit threshold.

Default behavior:

- dry-run (no launch command execution)
- derives VAI parameters from committed USD reserve:
  - `exogenousReserveInit = floor(committedUSD * reserveRatioBips / 10000)`
  - `vaiDebtCeiling = floor(exogenousReserveInit * 10000 / backingFloorBips)`
  - `vaiEpochMintLimit = max(1, floor(vaiDebtCeiling * epochMintLimitBips / 10000))`

Quick start:

```bash
cd scripts
cp launchpad.commits.template.json launchpad.commits.json
npm run launchpad:genesis
```

Execution mode (threshold met + real launch command):

```bash
cd scripts
TARGET_COMMITS_USD=1000000 \
LAUNCHPAD_DRY_RUN=false \
LAUNCH_COMMAND="node create-chain-on-subnet.mjs" \
npm run launchpad:genesis
```

The script writes:

- frozen launch decision artifact: `evidence-bundles/launchpad/launchpad-freeze-*.json`
- state marker: `evidence-bundles/launchpad/launchpad.state.json`
- generated launch genesis: `genesis.launchpad.json`

## Economic Flywheel Audit

Use this to produce runtime pass/fail evidence for treasury -> VAI -> UniV2 -> Keep3r launch behavior:

```bash
cd scripts
npm run audit:flywheel
```

Artifacts:

- `evidence-bundles/flywheel-audit/flywheel-*/flywheel-audit.json`
- `evidence-bundles/flywheel-audit/flywheel-*/flywheel-audit.md`
- `evidence-bundles/flywheel-audit/latest.txt`

Economic coherence audit (supply/risk math + liquidity depth realism):

```bash
cd scripts
npm run audit:economics
```

Artifacts:

- `evidence-bundles/economic-coherence/econ-*/economic-coherence.json`
- `evidence-bundles/economic-coherence/econ-*/economic-coherence.md`
- `evidence-bundles/economic-coherence/latest.txt`

VM/privacy mechanism audit (proof gate, fail-close, malformed rejection, timeout, backup takeover):

```bash
cd scripts
npm run audit:vm:privacy
```

Artifacts:

- `evidence-bundles/vm-privacy-audit/vmprivacy-*/vm-privacy-audit.json`
- `evidence-bundles/vm-privacy-audit/vmprivacy-*/vm-privacy-audit.md`
- `evidence-bundles/vm-privacy-audit/latest.txt`

Liquidity depth adjuster (targets reference-trade slippage bound):

```bash
cd scripts
npm run liquidity:deepen
```

Artifacts:

- `evidence-bundles/liquidity-depth/liquidity-*/liquidity-depth.json`
- `evidence-bundles/liquidity-depth/latest.txt`

Admin ownership rotation (clears temporary-admin blocker before launch):

```bash
cd scripts
npm run admin:rotate -- --new-owner <MULTISIG_OR_HSM_EOA>
```

Execute mode (submits transactions):

```bash
cd scripts
npm run admin:rotate -- --new-owner <MULTISIG_OR_HSM_EOA> --execute
```

Artifacts:

- `evidence-bundles/admin-rotation/rotation-*/ownership-rotation.json`
- `evidence-bundles/admin-rotation/rotation-*/ownership-rotation.md`
- `evidence-bundles/admin-rotation/latest.txt`

## Native AVAX Pool Path

For AVAX liquidity rails, use wrapped AVAX through the companion EVM bridge path and settle on VeilVM with a dedicated wrapped-AVAX asset rail.

- Use C-Chain/EVM bridge rails for AVAX ingress/egress.
- Do not use P-Chain or X-Chain directly as AMM execution venues.
- Keep VEIL/VAI risk invariants enforced on VeilVM even when AVAX liquidity is bridged in.

## ZK Flow (v1)

1. Prover computes `public_inputs_hash = sha256("VEIL_CLEAR_V1" || marketID || windowID || clearPrice || totalVolume || fillsHashLen || fillsHash)`.
2. Prover submits `SubmitBatchProof` with:
   - `public_inputs_hash` (32 bytes)
   - `fills_hash`
   - full proof blob (stored in Vellum proof storage)
3. `SubmitBatchProof` requires:
   - governance-configured prover authority + proof deadlines
   - if verifier is enabled: proof validity check before record is written
4. `ClearBatch` is fail-closed and requires:
   - `RevealBatch` share count for the target `windowID` meets governance threshold (`RevealThresholdX/RevealThresholdY`, default `13/20`)
   - each `RevealBatch` signer matches configured committee member for that `validator_index`
   - each `RevealBatch` carries `envelope_epoch` + `envelope_committee_key_id` that must match per-window batch envelope metadata captured at `CommitOrder` time
   - proof record exists and matches required proof type/deadline
   - proof blob exists in Vellum store and hashes to stored commitment
   - stored `public_inputs_hash` matches canonical hash computed from clear inputs
   - if verifier is enabled: cryptographic proof verification must pass

RPC helpers:
- `clearinputshash`: compute canonical public-input hash
- `batchproof`: get batch proof metadata
- `vellumproof`: get stored proof blob
- `bloodsworn`: read validator trust profile (non-consensus metadata)
- `glyph`: read proof-derived inscription metadata (non-consensus metadata)

## Verifier Activation

Runtime env toggles:

- `VEIL_ZK_VERIFIER_ENABLED=true`
- `VEIL_ZK_VERIFIER_STRICT=true` (optional fail-close when verifier/key unavailable)
- `VEIL_ZK_GROTH16_VK_PATH=/path/to/groth16.vk` (if `requiredProofType=1`)
- `VEIL_ZK_PLONK_VK_PATH=/path/to/plonk.vk` (if `requiredProofType=2`)
- `VEIL_ZK_REQUIRED_CIRCUIT_ID=clearhash-v1|shielded-ledger-v1` (optional hard gate)

`VEIL_ZK_VERIFIER_ENABLED=true` requires at least one matching VK path to load at startup.

Proof blob format:

- Legacy mode: raw proof bytes (accepted only when verifier mode is disabled)
- Envelope mode (`VZK1`): `magic(4) | proof_type(1) | proof_len(4) | witness_len(4) | proof | public_witness`
- Envelope mode (`VZK2`): `magic(4) | proof_type(1) | circuit_len(1) | proof_len(4) | witness_len(4) | circuit_id | proof | public_witness`

When verifier mode is enabled, an envelope with public witness is required.
`VEIL_ZK_REQUIRED_CIRCUIT_ID` enforces circuit identity at consensus verification time.

## Glyphs and Bloodsworn

- Every accepted `SubmitBatchProof` mints one deterministic `Glyph` from proof metadata and tx entropy.
- Every accepted proof also updates the prover's `Bloodsworn` profile (`total_accepted_proofs`, `active_streak`).
- These systems are explicitly non-consensus-critical:
  - no block validity rules depend on `Glyph` rarity/class
  - no token mint or economic reward is attached by default
  - proof-gating security remains unchanged (commitment + input-hash checks in `ClearBatch`)

## Build

```bash
go build ./...
```

Generate Groth16 fixture keys/proof envelope for verifier rollout:

```bash
go run ./cmd/veilvm-zktool -out ./zk-fixture
```

Generate shielded-ledger fixture artifacts:

```bash
go run ./cmd/veilvm-zktool -circuit shielded-ledger-v1 -out ./zk-fixture-shielded
```

`veilvm-zktool` defaults to `clearhash-v1` and can emit either circuit fixture set:
- clear-hash: `groth16_clearhash_pk.bin` / `groth16_clearhash_vk.bin`
- shielded-ledger: `groth16_shielded_ledger_pk.bin` / `groth16_shielded_ledger_vk.bin`

Run zkbench with real Groth16 proofs (fixture mode):

```bash
PROOF_MODE=groth16 GROTH16_PK_PATH=./zk-fixture/groth16_clearhash_pk.bin CHAIN_ID=<CHAIN_ID> go run ./cmd/veilvm-zkbench
```

Shielded-ledger fixture mode (current local launch-gate path):

```bash
PROOF_MODE=groth16 PROOF_CIRCUIT_ID=shielded-ledger-v1 GROTH16_PK_PATH=./zk-fixture-new/groth16_shielded_ledger_pk.bin CHAIN_ID=<CHAIN_ID> go run ./cmd/veilvm-zkbench
```

Fee safety controls for long bench runs:

- `REFUEL_PRIVATE_KEY`: optional secondary key used to auto-top-up the bench key when fee balance drops.
- `GAS_SAFETY_BPS` (default `13000`): multiplies projected run fee budget (130% default).
- `GAS_RESERVE` (default `250000`): minimum fee buffer to keep before each tx.
- `REFUEL_AMOUNT` (default `5000000`): minimum transfer size used during auto-refuel.
- `STRICT_FEE_PREFLIGHT` (default `false`): when `true`, fail fast on startup/per-tx fee preflight checks even without refuel configured.

`veilvm-zkbench` now performs:

1. Startup fee preflight (strict fail only with `STRICT_FEE_PREFLIGHT=true` or when refuel is configured).
2. Runtime insufficient-fee detection on each action submit.
3. One-shot auto-refuel + retry on insufficient-fee submission errors.

## Run

```bash
go run ./cmd/veilvm
```

Run the VEIL order router (direct + EVM order/liquidity intent relay):

```bash
ORDER_ROUTER_PRIVATE_KEY=<REDACTED> ORDER_ROUTER_RELAY_SECRET=<shared-secret> go run ./cmd/veilvm-order-router
```

Router endpoints:

- `POST /orders`: direct order ingest
- `POST /intents/native/execute`: native opaque order intent ingest
- `POST /intents/native/liquidity/execute`: native opaque liquidity intent ingest
- `POST /evm/intents/execute`: relayer-only EVM intent ingest (`x-relay-secret` required)
- `POST /evm/liquidity/execute`: relayer-only EVM UniV2 liquidity ingest (`x-relay-secret` required)
- `GET /health`: router health

Router privacy control env:

- `ORDER_ROUTER_ENABLE_EVM_INGRESS=true|false` (default `true`)
- `ORDER_ROUTER_NATIVE_INTENT_SECRET=<optional native ingress secret>`
- `ORDER_ROUTER_REQUIRE_OPAQUE_ENVELOPE=true|false` (default `true`)
- `ORDER_ROUTER_MIN_OPAQUE_ENVELOPE_BYTES=<N>` (default `96`)

EVM intent bridge components:

- order contract: `companion-evm/contracts/VeilOrderIntentGateway.sol`
- liquidity contract: `companion-evm/contracts/VeilLiquidityIntentGateway.sol`
- companion flow doc: `companion-evm/README.md`
- operations runbook: `VEIL_EVM_INTENT_RELAY_RUNBOOK.md`
- upgrade checklist: `OPAQUE_EVM_INTENT_MIGRATION_CHECKLIST.md`
- relay helper: `scripts/relay-intent-to-router.mjs`
- liquidity relay helper: `scripts/relay-liquidity-intent-to-router.mjs`
- native order intent helper: `scripts/submit-native-intent-to-router.mjs`
- native liquidity intent helper: `scripts/submit-native-liquidity-intent-to-router.mjs`
- automated opaque relay worker: `scripts/relay-opaque-intents.mjs`
- mailbox template for commit-only submits: `scripts/intent-mailbox.template.json`

Forward one EVM intent into VEIL:

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<shared-secret> npm run relay:intent -- \
  --intent-id 0x<intent-id> \
  --market-key 0x<market-key> \
  --envelope 0x<opaque-bytes> \
  --commitment 0x<sha256-envelope> \
  --nullifier 0x<bytes32> \
  --market-type polygon_native \
  --routing-fee-bps 3 \
  --source-tx-hash 0x<evm-tx-hash>
```

Forward one EVM liquidity intent into VEIL (UniV2 rail):

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<shared-secret> npm run relay:liquidity-intent -- \
  --intent-id 0x<intent-id> \
  --envelope 0x<opaque-bytes> \
  --commitment 0x<sha256-envelope> \
  --nullifier 0x<bytes32> \
  --operation add_liquidity \
  --asset0 0 \
  --asset1 1 \
  --amount0 1000000 \
  --amount1 2000000 \
  --min-lp 1 \
  --source-tx-hash 0x<evm-tx-hash>
```

Run automated opaque relayer scan/watch:

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<shared-secret> EVM_RELAY_EXECUTOR_PRIVATE_KEY=<REDACTED> VEIL_INTENT_MAILBOX_PATH=./intent-mailbox.json \
  npm run relay:opaque -- --from-block <START_BLOCK>

ORDER_ROUTER_RELAY_SECRET=<shared-secret> EVM_RELAY_EXECUTOR_PRIVATE_KEY=<REDACTED> VEIL_INTENT_MAILBOX_PATH=./intent-mailbox.json \
  npm run relay:opaque:watch -- --from-block <START_BLOCK> --poll-ms 5000

npm run check:opaque-intent-privacy
```

Commit-only privacy note:

- Companion gateways submit only `commitment` + `nullifier` on-chain; envelope bytes must be delivered off-chain to relayer mailbox/transport.
- VM-native path does not require companion EVM contracts. Set `ORDER_ROUTER_ENABLE_EVM_INGRESS=false` to run VEIL-only ingress.
- Router rejects plaintext/JSON envelopes by default (`ENVELOPE_PRIVACY_REJECTED`); use opaque/randomized bytes and keep routing/liquidity hints in relay JSON or mailbox hint fields.

Submit native opaque order intent:

```bash
cd scripts
npm run native:intent -- \
  --intent-id 0x<intent-id> \
  --market-id <market-id-or-key> \
  --envelope 0x<opaque-bytes> \
  --commitment 0x<sha256-envelope> \
  --nullifier 0x<bytes32> \
  --native-network veil \
  --routing-fee-bps 0
```

Submit native opaque liquidity intent:

```bash
cd scripts
npm run native:liquidity-intent -- \
  --intent-id 0x<intent-id> \
  --envelope 0x<opaque-bytes> \
  --commitment 0x<sha256-envelope> \
  --nullifier 0x<bytes32> \
  --operation add_liquidity \
  --asset0 0 \
  --asset1 1 \
  --amount0 1000000 \
  --amount1 2000000 \
  --min-lp 1
```

Liquidity private-gate behavior:

- under the private admission gate, liquidity requests are committed as `CommitOrder` envelopes.
- direct public AMM actions (`AddLiquidity`, `SwapExactIn`, `RemoveLiquidity`) are rejected at admission.
- liquidity fields above are relay/context hints; on-chain object is the opaque commitment envelope.

UniV2 asset IDs:

- `0 = VEIL`
- `1 = VAI`

Current relay model:

- VeilVM execution actor is the router signer key (`ORDER_ROUTER_PRIVATE_KEY`).

If relaying `create_pool`, configure governance signer:

```bash
ORDER_ROUTER_GOVERNANCE_PRIVATE_KEY=<REDACTED>
```

### Sync Guardian (VEILvm + Companion EVM)

Use sync guardian to keep ingress fail-closed unless:

- VEILvm nodes are alive, advancing, and within configured drift.
- Companion EVM instances agree on chain/head within configured drift.
- Router health is up on the expected VEIL chain ID.

Run once (strict gate):

```bash
cd scripts
npm run sync:guardian
```

Run continuously:

```bash
cd scripts
npm run sync:guardian:watch
```

Config template:

- `scripts/sync-guardian.config.template.json`

Outputs:

- status json: `examples/veilvm/.runtime/sync-guardian.status.json`
- lock file (ingress closed): `examples/veilvm/.runtime/sync-guardian.lock`

## Local Ops

### 60-second restart wizard (Windows)

Fast path:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\install-wizard.ps1
```

Automatic repair path (if readiness misses target):

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\install-wizard.ps1 -Repair
```

Detailed failure log + root causes:

- `RESTART_PATH_TROUBLESHOOTING_2026-02-20.md`

Mainnet bridge + Chainlink connectivity gate:

```bash
cd scripts
npm run bridge:activate-mainnet -- --config ./mainnet-bridge.config.template.json
npm run bridge:check-mainnet-live -- --config ./mainnet-bridge.config.template.json
```

Security audit harness (`evmbench`) quickstart:

```powershell
cd scripts
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-up.ps1
$env:OPENAI_API_KEY: <REDACTED>
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-audit.ps1 -FailOnFindings
```

Audit evidence output root:

- `evidence-bundles/audit-closure`

Security defaults in `docker-compose.local.yml`:

- admin API disabled (`--api-admin-enabled=false`)
- allowed hosts restricted to local callers (`localhost`, `127.0.0.1`, `host.docker.internal`)
- RPC and staking ports exposed only on localhost (`127.0.0.1`)
- verifier env is strict-enabled with required circuit id `shielded-ledger-v1` for the default local profile

Override the local verifier gate to shielded-ledger fixture keys:

```powershell
# generate shielded-ledger fixture artifacts if not already present
go run ./cmd/veilvm-zktool -circuit shielded-ledger-v1 -out ./zk-fixture-shielded
Copy-Item ./zk-fixture-shielded/groth16_shielded_ledger_vk.bin ./zk-fixture-new/groth16_shielded_ledger_vk.bin -Force

$env:VEIL_ZK_REQUIRED_CIRCUIT_ID="shielded-ledger-v1"
$env:VEIL_ZK_GROTH16_VK_PATH="/root/.avalanchego/zk/groth16_shielded_ledger_vk.bin"
docker compose -f docker-compose.local.yml up -d --build
```

`G3` shielded-ledger circuit scope evidence is archived at `evidence-bundles/zk-circuit-assurance/latest.txt`.

Recurring local ops issues (and fast recovery):

1. Docker daemon hangs / CLI timeouts.
   - Symptom: `docker version` or `docker ps` stalls/timeouts.
   - Recovery: restart Docker Desktop, then re-run the compose command.
2. Wrong stack responds on `9650` while VeilVM `9660` is down.
   - Symptom: evidence runner falls back to `http://127.0.0.1:9650` and health stays `503`.
   - Recovery: bring up VeilVM node explicitly:
     - `docker compose -f docker-compose.local.yml up -d --build node`
3. Verifier circuit gate resets away from `shielded-ledger-v1` after container restart.
   - Symptom: `proof circuit mismatch` during shielded-ledger evidence runs.
   - Recovery:
     - `$env:VEIL_ZK_REQUIRED_CIRCUIT_ID="shielded-ledger-v1"`
     - `$env:VEIL_ZK_GROTH16_VK_PATH="/root/.avalanchego/zk/groth16_shielded_ledger_vk.bin"`
     - `docker compose -f docker-compose.local.yml up -d --build node`
4. Readiness stays `503` right after node restart.
   - Symptom: health payload reports bootstrapping not finished.
   - Recovery: wait until `http://127.0.0.1:9660/ext/health/readiness` returns `200`, then start evidence runs.
5. After `docker compose down -v`, VEIL chain is missing.
   - Symptom: evidence runner reports `no VEIL VM chains found`.
   - Recovery:
     - `cd scripts && node setup-local.mjs` (recreates subnet + VEIL chain)
     - update `--track-subnets=<new subnetID>` in `docker-compose.local.yml`
     - restart node: `docker compose -f docker-compose.local.yml up -d --build node`
     - rerun with chain auto-discovery (omit `--chain-id`) or pass the new chain ID.

Launch-gate evidence runner:

```powershell
cd scripts
npm.cmd run evidence:preflight
npm.cmd run evidence:launch-gates
```

Rapid iteration gate mode (fastest feedback):

```powershell
cd scripts

# Gate set A: fuel + shielded smoke
node run-launch-gate-evidence.mjs --only fuel,smoke --batch-size 1 --windows-per-size 1

# Gate set B: remaining checks (no smoke rerun)
node run-launch-gate-evidence.mjs --only backup,negative,malformed,timeout --skip-fuel-preflight --batch-size 1 --windows-per-size 1 --timeout-batches 1
```

Supported gate selectors:

- `--only fuel,smoke,backup,negative,malformed,timeout`
- `--skip-smoke`
- Existing `--skip-*` flags still work and can be combined with `--only`.

Latest saved evidence artifacts:

- `evidence-bundles/latest-launch-gate-evidence.txt`
- `evidence-bundles/saved/latest-launch-gate-evidence.txt`
- `evidence-bundles/saved/20260219-184441-launch-gate-evidence.zip`

Smoke test:

```bash
# validate an existing chain
node scripts/smoke-local.mjs --chain-id <CHAIN_ID>

# validate full setup flow (CreateSubnet + CreateBlockchain)
# for fresh-volume testing, wipe docker volumes first
node scripts/smoke-local.mjs --run-setup
```

Legacy native AMM liquidity movement smoke (permissive mode only):

```powershell
# from repo root: rebuild node so latest RPC/action fixes are live
docker build -t veilvm-node:latest -f examples/veilvm/Dockerfile.node .
cd examples/veilvm
docker compose -f docker-compose.local.yml up -d --force-recreate node

# from repo root: run VM-native liquidity smoke through Docker Go toolchain
cd ../..
docker run --rm `
  -e NODE_URL=http://host.docker.internal:9660 `
  -e CHAIN_ID=<CHAIN_ID> `
  -v "${PWD}:/workspace" `
  -w /workspace/examples/veilvm `
  golang:1.23-bullseye `
  bash -lc 'export PATH=/usr/local/go/bin:$PATH; go run ./cmd/veilvm-smoke'
```

Expected success markers in JSON output (only when running a permissive policy profile):

- `pass: true`
- steps include:
  - `ensure_pool`
  - `mint_vai`
  - `add_liquidity`
  - `swap_veil_to_vai`
  - `remove_liquidity`
  - `verify_remove_liquidity`

If you see `can't find method "veilvm.Lpbalance"`:

- the running node image is stale; rebuild `veilvm-node:latest` and force-recreate the node container again.

If you see `proof-verified private action required`:

- this is expected under the default private-only gate.
- use the private liquidity intent path (`/intents/native/liquidity/execute`) and verify `CommitOrder` tx type.

Create a fresh VEIL chain on the already-tracked subnet (no DB wipe):

```bash
cd scripts
node create-chain-on-subnet.mjs
```

Dry-run preflight (no tx submission):

```bash
cd scripts
npm run create:chain:dry-run
```

Remote/AVAcloud-style execution (explicit signer + optional RPC headers):

```bash
cd scripts
NODE_URL="https://<your-avax-node-rpc>" \
SUBNET_ID="<existing-subnet-id>" \
P_CHAIN_PRIVATE_KEY=<REDACTED> \
P_CHAIN_ADDRESS="P-..." \
RPC_HEADERS_JSON='{"x-api-key":"<api-key-if-needed>"}' \
node create-chain-on-subnet.mjs --dry-run
```

Reusable templates:

- `scripts/create-chain-on-subnet.env.template`
- `scripts/create-chain-on-subnet.uup.template.ps1`

PowerShell UUP quick-start:

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm\scripts
Copy-Item .\create-chain-on-subnet.uup.template.ps1 .\create-chain-on-subnet.uup.ps1
notepad .\create-chain-on-subnet.uup.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\create-chain-on-subnet.uup.ps1
```

When dry-run output is healthy (`utxoCount > 0`), remove `--dry-run` to issue `CreateBlockchain`.

## Protocol Docs

- `VEIL_MASTER_RUNBOOK.md` - execution order and launch critical path
- `CRITICAL_PHASE_CONTROL_TOWER.md` - critical-phase operator sequence, artifact gates, and signature policy caveat
- `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md` - hard production go/no-go gates with owners and evidence requirements
- `VEIL_V1_NATIVE_PRIVACY_SPEC.md` - whitepaper-aligned protocol requirements
- `VEIL_EXECUTION_PACKAGE.md` - tokenomics, COL, risk controls, launch gates
- `VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md` - bridge/EVM primitive rollout checklist and pass conditions
- `VEIL_MAINNET_CHAINLINK_BRIDGE_RUNBOOK.md` - Avalanche mainnet + Chainlink live bridge activation and verification
- `EVMBENCH_AUDIT_RUNBOOK.md` - evmbench bootstrap, run commands, and G8 closure evidence workflow
- `VEIL_WHITEPAPER_ALIGNMENT_MATRIX.md` - claim-by-claim whitepaper alignment checklist
- `VEIL_HANDOFF_2026-02-19.md` - project-wide handoff snapshot and immediate next actions
- `SAVEPOINT_2026-02-21_PRIVATE_LIQUIDITY.md` - current private-liquidity execution checkpoint

