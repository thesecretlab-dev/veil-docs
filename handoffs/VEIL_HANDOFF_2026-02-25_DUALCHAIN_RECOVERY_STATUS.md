# VEIL Handoff 2026-02-25: Dual-Chain Recovery Status

Status: ACTIVE RECOVERY  
Timestamp: 2026-02-25 11:01:45 -05:00  
Scope: VEILVM + companion VEILPOS mainnet runtime and onboarding readiness gates

## 1) Executive Status

- Overall onboarding readiness is `NO-GO`.
- Latest readiness bundle:
  - `C:\Users\Josh\hypersdk\examples\veilvm\evidence-bundles\dualchain-readiness\dualchain-20260225-054055\readiness.md`
- Current gate board:
  - `DC0 PASS`
  - `DC1 PASS`
  - `DC2 FAIL`
  - `DC3 PASS`
  - `DC4 PASS`
  - `DC5 PASS`
  - `DC6 FAIL`
  - `DC7 FAIL`

## 2) Live Runtime Snapshot

- VEILVM readiness:
  - `http://127.0.0.1:9660/ext/health/readiness = 200`
- Companion health:
  - `http://127.0.0.1:9650/ext/health = 503`
  - `http://127.0.0.1:9652/ext/health = 503`
- Running node IDs:
  - companion A (`9650`): `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES`
  - companion B (`9652`): `NodeID-NgTGj8XBitUPmSdwSRzEhWeB7XMADY5pA`
  - VEILVM primary (`9660`): `NodeID-G2K9Jd6Tm4robALQsZVfdCmbZpFuWtCUx`
  - VEILVM secondary (`9662`): `NodeID-LyiiaRJCAk3SvvR67MMRt7AAUxQWXRxYr`

## 3) Root Cause (DC2)

VEILPOS validator set on mainnet P-Chain still has:

- `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` weight `100` (legacy/disconnected)
- `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES` weight `1` (connected)

Connected-stake threshold is effectively blocked by stake geometry:

- target connected ratio: `>= 80%`
- with disconnected `100`, required connected weight is `>= 400`
- current connected weight is `1`
- additional connected weight needed: `399`

Contract-level constraints already confirmed:

- removing `D26` currently reverts with `MaxChurnRateExceeded(100)` (20% churn cap)
- removing `J4V` reverts with `MinStakeDurationNotPassed(...)`

## 4) Funding Reality

`veil-funded` current balances:

- C: `149,812,121 nAVAX` (`0.149812121 AVAX`)
- P: `194,856 nAVAX` (`0.000194856 AVAX`)
- X: `4,000,000 nAVAX` (`0.004 AVAX`)

PoS manager settings in effect include:

- `minimumStakeAmount = 1 AVAX`
- `weightToValue(1) = 1 AVAX`
- `maximum churn percentage = 20`

Implication:

- no new validator registration can be initiated yet (C balance < 1 AVAX)
- even after crossing 1 AVAX, full dilution path without `D26` recovery is about `399 AVAX` additional connected stake

## 5) Secondary Blocker (DC6)

Companion stable-token policy is still legacy mode:

- file: `C:\Users\Josh\hypersdk\examples\veilvm\scripts\companion-evm.addresses.json`
- current:
  - `"wVaiToken": ""`
  - `"vaiOrigin": "companion-native-legacy"`

Required for `DC6` pass:

- set bridge-origin token address in `wVaiToken`
- set `vaiOrigin` to `veilvm-bridge`

## 6) What Was Completed In This Window

- Verified live endpoints, node IDs, and validator set.
- Reconfirmed blocker errors from manager paths and churn limits.
- Consolidated locally available AVAX into `veil-funded` C-chain where possible.
- Regenerated readiness package:
  - `dualchain-20260225-054055`
- Updated canonical docs:
  - `LIVE_DEVLOG.md`
  - `VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md`
  - `VEIL_GEMINI_SIMULATION_MODEL_INPUT_2026-02-25.md`

## 7) Next Operator Actions (Ordered)

1. Resolve connected-stake deadlock by choosing one path:
   - Path A: recover/run the missing `NodeID-D26...` validator identity.
   - Path B: fund `veil-funded` C-chain sufficiently to add ~`399` connected weight.
2. After stake path is resolved, re-check companion health/readiness until `DC2` flips.
3. Migrate stable-token policy to bridge-origin (`wVaiToken` + `vaiOrigin=veilvm-bridge`) to clear `DC6`.
4. Re-run:
   - `powershell -NoProfile -ExecutionPolicy Bypass -File C:\Users\Josh\hypersdk\examples\veilvm\scripts\build-dualchain-readiness-package.ps1`
5. Confirm `DC0..DC7` all pass before claiming onboarding-ready.

## 8) Mainnet-Only Guardrail

- Keep companion operations strictly mainnet.
- Do not use Fuji/testnet flags or fallback subnet defaults in active recovery.
