# VEIL Ecosystem Simulation Model Input (Gemini)

Generated: 2026-02-25
Scope: full dual-chain VEIL ecosystem simulation baseline (runtime + economics + routing + validator policy + readiness gates)
Snapshot source: local workspace runtime + latest readiness bundle (`dualchain-20260224-221637`)

## 1) System Summary

VEIL is a dual-chain system:

1. VEILVM (HyperSDK VM chain):
   - Native intent execution and privacy-oriented order/liquidity envelope flow.
   - Validator policy source of truth: stake VEIL to hold vVEIL (validator gating condition).
2. Companion EVM chain (VEILPOS Subnet-EVM):
   - EVM contracts, bridge interfaces, and liquidity/economic surfaces (Treasury, Keep3r, UniV2, token rails).
3. Avalanche Mainnet:
   - Funding rails (C/P/X), Teleporter references, validator-manager anchor context.

Design intent:
- Validator participation semantics are anchored on VEILVM stake -> vVEIL.
- Companion is execution/economic EVM plane.
- Onboarding readiness is strictly evidence-driven via DC0..DC7.

## 2) Current Runtime Snapshot

Observed healthy:
- VEILVM node health/readiness (`9660`) = 200/200
- VEIL order router (`9098`) = 200
- Native and EVM router endpoints accept test intent/liquidity payloads when correctly authenticated.

Observed not yet healthy:
- Companion node A (`9650`) health/readiness currently 503 (bootstrapping)
- Companion node B (`9652`) stopped in A-first fast-path mode
- `sigagg` and `sigagg-proxy` stopped in A-first fast-path mode

Auto-finalize status:
- `auto-dualchain-finalize.ps1 -BootstrapFastPath` is running and tracking bootstrap progress.
- Latest seen progress: companion A around mid-40% bootstrap completion.

## 3) Chain Metadata

### 3.1 VEILVM
- Chain ID: `2BzfxWHaDFn6nWuysYJXuswJqqMsChutq19sZhESNF3brTjK3n`
- Subnet ID: `2f97zdkH3s66Ar21UiHzagvUYCjJd3TggPoQmChtmB7kuwHNgH`
- VEIL API URL:
  - `http://127.0.0.1:9660/ext/bc/2BzfxWHaDFn6nWuysYJXuswJqqMsChutq19sZhESNF3brTjK3n/veilapi`

### 3.2 Companion EVM (VEILPOS)
- Subnet ID: `hrQA48Fj1TG9h4NVb8kFhZjC6P4RwvCwgioyC8wmqSfoZ5GmB`
- Blockchain ID: `hZiEpoLLNh7twd9sEiCuV1kMgNvNoveGvjM8QhgnBwoxUajD2`
- EVM chainId: `22209`
- RPC URL:
  - `http://127.0.0.1:9650/ext/bc/hZiEpoLLNh7twd9sEiCuV1kMgNvNoveGvjM8QhgnBwoxUajD2/rpc`
- Subnet-EVM config highlights:
  - `warpConfig.requirePrimaryNetworkSigners=true`
  - `warpConfig.quorumNumerator=67`
  - `feeConfig.targetBlockRate=2`
  - `feeConfig.gasLimit=8000000`

### 3.3 Avalanche Mainnet Anchors
- Mainnet RPC: `https://api.avax.network/ext/bc/C/rpc`
- Teleporter registry: `0x7C4360...8d98`
- Teleporter messenger: `0x253b27...5fcf`

## 4) Service Topology and Ports

VEIL runtime:
- `veilvm-node` -> host `127.0.0.1:9660 -> container 9650`
- `veilvm-node-secondary` -> host `127.0.0.1:9662 -> container 9650`
- `veilvm-order-router` -> host `127.0.0.1:9098`

Companion runtime:
- `companion-evm-node-a` -> host `127.0.0.1:9650`
- `companion-evm-node-b` -> host `127.0.0.1:9652`
- `sigagg` -> host `127.0.0.1:9090`
- `sigagg-proxy` -> host `127.0.0.1:19090`

Companion compose critical env dependency:
- `COMPANION_TRACK_SUBNET` is required (`--track-subnets=...`)
- `COMPANION_BLOCKCHAIN_ID` is required to install chain config into node volumes

## 5) Router Interfaces (for Simulation)

Native endpoints (no relay secret header):
- `POST /intents/native/execute` on `http://127.0.0.1:9098`
- `POST /intents/native/liquidity/execute` on `http://127.0.0.1:9098`

EVM endpoints (must include `x-relay-secret`):
- `POST /evm/intents/execute` on `http://127.0.0.1:9098`
- `POST /evm/liquidity/execute` on `http://127.0.0.1:9098`

Relay auth:
- Current local relay secret configured in restart flow: `local-dev-relay`

Envelope semantics:
- `envelope` payload must hash to provided `commitment` via SHA-256.
- Mismatch triggers `COMMITMENT_MISMATCH`.

## 6) Companion Contract and Role Map

From `scripts/companion-evm.addresses.json`:

Core role EOAs:
- temp admin: `0x580358...503A`
- relayer1: `0x7deFD0...Ad53`
- relayer2: `0x7843eb...4bdF`
- ops keeper: `0x698acA...4A55`
- deployer1: `0x9d0FE6...B1b4`

Gateway contracts:
- orderIntentGateway: `0x0689A1...7da9`
- liquidityIntentGateway: `0xeC0c92...fbDe`

Token and economics:
- wVEIL: `0x7c93F4...8116`
- VAI token: `0x05e0c4...F2Eb`
- VeilTreasury: `0x937894...EED6`
- VeilKeep3r: `0xc093Cb...BbFc`
- UniV2 factory: `0x66e87C...3a81`
- UniV2 router: `0xe0Ea56...9cf1`
- UniV2 pair (wVEIL/stable): `0x39b8Ee...95db`

Precompile/system refs:
- nativeMinter: `0x020000...0001`
- txAllowList: `0x020000...0002`
- contractDeployerAllowList: `0x020000...0000`
- feeConfigManager: `0x020000...0003`

## 7) Token-Origin Policy Model

Supported modes in deploy tooling:
- `veilvm-bridge` (target policy)
- `companion-native-legacy` (legacy fallback)

Current state:
- `wVaiToken=""`
- `vaiOrigin="companion-native-legacy"`

Implication:
- Companion stable-token policy gate fails until bridged-origin `wVAI` is set and origin mode is switched to `veilvm-bridge`.

## 8) Validator Semantics Model (Critical)

Policy manifest:
- `scripts/dualchain-validator-stake.manifest.json`
- Policy name: `dual_chain_validator_requires_veil_stake`

Current validators in manifest:
- `NodeID-BRWmyj4aQPgx1suA3Le9km1aF6sQnmVyw` (min vVEIL raw: 1)
- `NodeID-AvwgLY7XaaisHondANg4MTB58WV5StVgD` (min vVEIL raw: 1)

Model rule:
- Network validation eligibility should be represented as requiring VEIL stake on VEILVM that materializes as vVEIL threshold satisfaction.
- Companion validator semantics should not supersede VEILVM stake semantics; treat companion as execution plane.

## 9) Funding State (Mainnet Rail)

Latest observed:
- Deploy key C: `0.011274621003629639 AVAX`
- Deploy key P: `0.012785368 AVAX`
- Deploy key X: `0.004 AVAX`
- Relayer key C: `0.400999308951026 AVAX`

Model implication:
- Funding gate currently passes (non-zero on C/P/X for deploy key).

## 10) Readiness Gate Model (DC0..DC7)

Current gate statuses:
- `DC0` FAIL: dual runtime containers not all up
- `DC1` PASS: VEILVM health/readiness
- `DC2` FAIL: companion health/readiness
- `DC3` PASS: VEILPOS sidecar mainnet metadata present
- `DC4` PASS: deploy-key C/P/X funded
- `DC5` PASS: VEIL stake -> vVEIL validator checks
- `DC6` FAIL: stable token policy not bridge-origin
- `DC7` FAIL: overall onboarding gate (requires DC0..DC6)

Simulation should model DC7 as hard fail-closed.

## 11) End-to-End Flow to Simulate

Flow A: Native VEILVM intent/liquidity path
1. User submits VENC1 envelope to native order endpoint.
2. User submits VENC1 envelope to native liquidity endpoint.
3. Router commits on VEILVM.
4. Expected now: operational (already observed).

Flow B: EVM relay path through router
1. Relayer submits envelope + commitment + nullifier to `/evm/intents/execute` with relay secret.
2. Relayer submits envelope + commitment + liquidity fields to `/evm/liquidity/execute`.
3. Router validates secret and commitment hash.
4. Router emits accepted state and VEIL tx hash.
5. Expected now: operational at router level.

Flow C: Companion execution + econ layer
1. Companion reaches health/readiness 200 on A and B.
2. `sigagg` and `sigagg-proxy` online.
3. EVM scripts can read chain + execute liquidity and audit flows.
4. Stable token mode migrated to bridge-origin wVAI.
5. Expected after fixes: DC2 + DC6 pass.

## 12) Gaps for Full Ecosystem Operability

Hard gaps:
1. Companion bootstrap complete on node A, then node B + sigagg bring-up (DC2/DC0).
2. Bridge-origin stable token migration (`wVaiToken` populated, `vaiOrigin=veilvm-bridge`) (DC6).

Planned liquidity extension:
1. Existing deploy flow is centered on `wVEIL <-> stable` pool.
2. `wAVAX/VEIL` pool route is not yet first-class in current deploy script and should be explicitly added for complete liquidity routing design.

## 13) Minimal Machine-Readable Seed (Gemini)

```json
{
  "snapshot": {
    "runId": "dualchain-20260224-221637",
    "overallPass": false
  },
  "chains": {
    "veilvm": {
      "chainId": "2BzfxWHaDFn6nWuysYJXuswJqqMsChutq19sZhESNF3brTjK3n",
      "subnetId": "2f97zdkH3s66Ar21UiHzagvUYCjJd3TggPoQmChtmB7kuwHNgH",
      "veilApiUrl": "http://127.0.0.1:9660/ext/bc/2BzfxWHaDFn6nWuysYJXuswJqqMsChutq19sZhESNF3brTjK3n/veilapi"
    },
    "companion": {
      "subnetId": "hrQA48Fj1TG9h4NVb8kFhZjC6P4RwvCwgioyC8wmqSfoZ5GmB",
      "blockchainId": "hZiEpoLLNh7twd9sEiCuV1kMgNvNoveGvjM8QhgnBwoxUajD2",
      "evmChainId": 22209,
      "rpcUrl": "http://127.0.0.1:9650/ext/bc/hZiEpoLLNh7twd9sEiCuV1kMgNvNoveGvjM8QhgnBwoxUajD2/rpc"
    }
  },
  "router": {
    "healthUrl": "http://127.0.0.1:9098/health",
    "relaySecret": "local-dev-relay",
    "nativeEndpoints": [
      "/intents/native/execute",
      "/intents/native/liquidity/execute"
    ],
    "evmEndpoints": [
      "/evm/intents/execute",
      "/evm/liquidity/execute"
    ]
  },
  "validatorPolicy": {
    "type": "dual_chain_validator_requires_veil_stake",
    "stakeAsset": "VEIL",
    "proofAsset": "vVEIL",
    "minVVEILRawPerValidator": "1"
  },
  "stableTokenPolicy": {
    "currentMode": "companion-native-legacy",
    "targetMode": "veilvm-bridge",
    "wVaiTokenConfigured": false
  },
  "gates": {
    "DC0": false,
    "DC1": true,
    "DC2": false,
    "DC3": true,
    "DC4": true,
    "DC5": true,
    "DC6": false,
    "DC7": false
  }
}
```

## 14) Update Override (2026-02-25 05:38 -05:00)

Use this section as the canonical override for current simulation state.

Latest readiness snapshot:

- run ID: `dualchain-20260225-054055`
- overall pass: `false`
- gates:
  - `DC0=true`
  - `DC1=true`
  - `DC2=false`
  - `DC3=true`
  - `DC4=true`
  - `DC5=true`
  - `DC6=false`
  - `DC7=false`

Current companion blocker geometry:

- Active VEILPOS validator weights:
  - disconnected bootstrap validator `NodeID-D26...` weight `100`
  - connected PoS validator `NodeID-J4V...` weight `1`
- Companion node B identity is `NodeID-NgTG...` and is not yet in active validator set.
- Native PoS manager constraints observed on-chain:
  - removing `D26` currently reverts with `MaxChurnRateExceeded(100)` (20% max churn limit).
  - removing `J4V` currently reverts with `MinStakeDurationNotPassed`.

Funding model correction:

- `minimumStakeAmount = 1 AVAX`
- `weightToValue(1) = 1 AVAX`
- `veil-funded` current C balance: `0.149812121294185682 AVAX`
- To satisfy 80% connected stake while disconnected weight remains `100`, connected weight must be at least `400`.
- Current connected weight is `1`.
- Required additional connected stake weight: `399` (approximately `399 AVAX`, plus gas), unless `NodeID-D26...` runtime is restored online.

Machine-readable override:

```json
{
  "snapshot": {
    "runId": "dualchain-20260225-054055",
    "overallPass": false
  },
  "gates": {
    "DC0": true,
    "DC1": true,
    "DC2": false,
    "DC3": true,
    "DC4": true,
    "DC5": true,
    "DC6": false,
    "DC7": false
  },
  "validatorSet": {
    "disconnectedWeight": 100,
    "connectedWeight": 1,
    "requiredConnectedWeightFor80Pct": 400,
    "additionalWeightRequired": 399
  },
  "staking": {
    "minimumStakeAvax": 1,
    "weightToValueAvaxPerWeight": 1,
    "veilFundedCAvax": 0.14981212129418568
  }
}
```
