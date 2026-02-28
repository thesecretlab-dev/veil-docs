# VEIL ANIMA Dual-Chain Validator Readiness Package

Status: Active  
Date: 2026-02-24  
Scope: incoming validator onboarding for ANIMA dual-chain runtime (VEILVM + companion EVM)

## 1) Hard Policy

For validator onboarding and new deploy operations, VEILVM and companion EVM must run in parallel.

- VEILVM-only bring-up is not considered onboarding-ready.
- Companion-only bring-up is not considered onboarding-ready.
- Readiness is evidence-driven and must be regenerated before each onboarding wave.

## 2) Operator Workflow

1. Start dual runtime in parallel:
   - `powershell -NoProfile -ExecutionPolicy Bypass -File <local-dev-path>`
2. Run validator setup/update flow on VEILVM:
   - `Set-Location <local-dev-path>`
   - `node setup-local.mjs`
3. Re-run parallel runtime startup after validator changes:
   - `powershell -NoProfile -ExecutionPolicy Bypass -File <local-dev-path>`
4. Generate the dual-chain readiness bundle:
   - `Set-Location <local-dev-path>`
   - `npm run readiness:dualchain`
5. Ensure validator stake manifest is maintained:
   - `<local-dev-path>`
   - Every dual-chain validator must map to a VEILVM address with required `minVVEILRaw`.

### Auto-Finalize (hands-off while companion bootstraps)

If companion bootstrap is still in progress, run:

- `Set-Location <local-dev-path>`
- `npm.cmd run ops:auto-finalize`

This command:

- applies A-first bootstrap fast-path (pauses node B + sigagg while node A catches up),
- waits for full dual-node companion health/readiness,
- runs VEIL native smoke + mainnet bridge live check,
- regenerates the dual-chain readiness package.

## 3) Generated Evidence Artifacts

The readiness generator writes:

- `<local-dev-path>`
- `<local-dev-path>`
- `<local-dev-path>`

## 4) Readiness Gates (Dual-Chain)

- `DC0` Dual Runtime Parallel Containers
  - Requires `veilvm-node`, `veilvm-node-secondary`, `companion-evm-node-a`, `companion-evm-node-b`, `sigagg`, and `sigagg-proxy` running.
- `DC1` VEILVM Health + Readiness
  - Requires `http://127.0.0.1:9660/ext/health=200` and `/ext/health/readiness=200`.
- `DC2` Companion EVM Health + Readiness
  - Requires node A (`9650`) and node B (`9652`) health/readiness endpoints to return `200`.
- `DC3` Companion Mainnet Sidecar Metadata
  - Requires `VEILPOS` sidecar to include `Networks.Mainnet`.
- `DC4` Mainnet Deploy Key Funding (C/P/X)
  - Requires non-zero AVAX for deploy key on C/P/X.
- `DC5` Dual-Chain Validator VEIL Stake -> vVEIL
  - Requires all validators in `scripts/dualchain-validator-stake.manifest.json` to meet `minVVEILRaw` via `veilvm.vveilbalance`.
- `DC6` Companion Stable Token Policy (wVAI Bridge-Origin)
  - Requires `scripts/companion-evm.addresses.json` to use `wVaiToken` and `vaiOrigin=veilvm-bridge`.
- `DC7` ANIMA Incoming Validator Onboarding Ready
  - Passes only when `DC0..DC6` all pass.

## 5) Definition of Done

Incoming ANIMA validator onboarding is considered ready only when:

1. Latest dual-chain readiness package exists.
2. `overall_pass=true` in readiness bundle.
3. No gate in `readiness.md` is `FAIL`.
4. Bundle path is shared in operator handoff and launch status docs.

## 6) Emergency Bypass

Bypass is allowed only for break-glass recovery:

- `restart-wizard.ps1 -SkipCompanion`

If bypass is used, onboarding readiness is automatically `NO-GO` until full dual-chain package is regenerated and passes.

## 7) Recovery Snapshot (2026-02-25)

Mainnet-only recovery work completed:

- `initializeValidatorSet` executed successfully on mainnet:
  - tx `0xde9da0...ec23`
- Companion validator registration for node A completed:
  - node ID `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES`
  - validation ID `Ty6pYrGkvC4jFRD2urZQFg6t6GzdRDSRn3EEBrDuH49cQpBZj`
  - P-chain tx `x4SKrBk41wyLsFv6g3t3g1uyWKvhhAzvGi7JzHH959cRKo2nT`
- Latest evidence package:
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-041722/readiness.md`

Remaining blockers to `DC7`:

- `DC2` still failing (companion A/B health/readiness not both `200`).
- Companion bootstrap validator `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` remains in active set with dominant weight.
- Force-removal path for `NodeID-D26...` currently fails at uptime-proof signature aggregation (insufficient live signing stake).
- Companion node B has been rebuilt and now runs as distinct `NodeID-NgTGj8XBitUPmSdwSRzEhWeB7XMADY5pA`, but additional validator registration is blocked by insufficient `veil-funded` stake balance for another minimum-weight PoS registration.

## 8) Recovery Update (2026-02-25 04:34 -05:00)

Latest readiness evidence:

- `evidence-bundles/dualchain-readiness/dualchain-20260225-043410/readiness.md`

Current gate board:

- `DC0 PASS`
- `DC1 PASS`
- `DC2 FAIL`
- `DC3 PASS`
- `DC4 PASS`
- `DC5 PASS`
- `DC6 FAIL`
- `DC7 FAIL`

Mainnet-only control-path outcomes during this update window:

- `changeWeight` is not allowed on this PoS manager (`weight can't be changed on Proof of Stake Validator Managers`).
- `addValidator` rejects large direct jumps (`desired validator weight ... exceeds max allowed weight change of 20`).
- Node B registration attempts at `weight=20` and `weight=1` still fail with `insufficient funds for transfer`.
- `removeValidator` for bootstrap validator `NodeID-D26...` remains blocked:
  - non-force path fails uptime retrieval (`503`),
  - force path fails threshold signature aggregation.

Validator set remains:

- `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` (`weight=100`)
- `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES` (`weight=1`)

## 9) Recovery Clarification (2026-02-25 05:37 -05:00)

Current state is now deterministic:

- Companion `503` is caused by validator connected-stake geometry, not by random process restarts.
- `NodeID-D26...` (weight `100`) is still active and disconnected.
- Local companion nodes are `NodeID-J4V...` and `NodeID-NgTG...`; no local node currently has `NodeID-D26...`.
- Native PoS manager confirms:
  - `D26` is not a PoS-owned staking-validator record (`owner=0x0` in staking-manager view).
  - `J4V` is PoS-owned by `veil-funded`, but removal is still gated by minimum stake duration.

Contract guardrails verified on-chain:

- Removing `D26` currently reverts with `MaxChurnRateExceeded(100)` (20% churn limit cannot remove 100 weight in one step).
- Registering additional PoS weight requires minimum `1 AVAX` stake per registration.

Required funding geometry for health recovery:

- Health target requires connected stake >= 80%.
- With disconnected `D26=100`, connected stake must be >= `400`.
- Current connected stake is `1`.
- Additional connected stake needed: `399`.
- Because `weightToValue(1)=1 AVAX`, this implies ~`399 AVAX` additional connected PoS stake (plus gas) unless `D26` runtime is restored online.

Latest evidence after local balance consolidation:

- readiness bundle: `evidence-bundles/dualchain-readiness/dualchain-20260225-054055/readiness.md`
- `veil-funded` C balance: `0.149812121294185682 AVAX` (still below the `1 AVAX` minimum stake for any new PoS registration).

