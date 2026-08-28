# VEIL stack (v1)

Status: Canonical  
Date: 2026-08-27  
Authority: this file + `veilvm/consts/types.go`. Older READMEs, ANIMA docs, and Feb evidence do not override it.

## One protocol

```
UX (veil-frontend)     Polymarket catalog. Native VeilVM local only. Not Fuji.
        │
ANIMA / ZER0ID / VEILdb     Agents, identity, off-chain state. They do not mint VEIL.
        │
Relayer                     EVM commitment/nullifier → VeilVM CommitOrder / liquidity commit
        │
┌───────┴────────────────────────────┐
VeilVM L1                            Companion EVM
HyperSDK                             Public rails only
chain app-id 22207                   Distinct EVM chainId (never reuse 22207)
native VEIL, native VAI              WVEIL wrap, Teleporter, intent gateways
commit-reveal, ZK, COL               wallets / explorers
```

VeilVM is the protocol. Companion EVM is an on-ramp. Solidity does not re-implement VAI, AMM, COL, or bonds.

## What each repo is allowed to be

| Repo | Job | Not its job |
|---|---|---|
| `veilvm` | Consensus, privacy, native VEIL/VAI, native AMM, COL | ERC-20s, MetaMask |
| `veil-contracts` | **v1 rails:** WVEIL, bridge minter, order/liquidity intent gateways, ZER0ID verifier, test faucet | Second VAI minter, second DEX, Maker DSS, Olympus, meme/404 |
| `veil-frontend` | UI. Until Fuji RPC exists, Polymarket is external data | Claiming native private markets are live |
| `zeroid` | ZK identity circuits + SDK | Chain economics |
| `anima` | Agent lifecycle against **v1 actions 0–18** | Submitting action IDs 19–41 |
| `veildb` | Off-chain stores | Issuing tokens |
| `veil-docs` | Specs and runbooks | Launch authority from Feb `PASS (local)` |

## Frozen v1 numbers

- Native actions: **19** (TypeIDs **0–18**). See `specs/VEIL_ACTION_REGISTRY.md`.
- IDs 19–41 (bonds, YRF/RBS, staking, oracle, bloodsworn, pause, reveal committee): **spec-only, not v1**. Do not advertise. Do not send on-chain.
- `TOTAL_SUPPLY = 990,999,000`. See `specs/VEIL_SUPPLY_FREEZE_2026-08-24.md`.
- **No Keep3r allocation.** Circulating is the 5% float. Cadence is relayer + operator.
- Lost owner `0xB9a05AFC8eff7eE6a84889Bb9C88A89eAA2f96af`: do-not-deploy-under.
- Companion `eth_chainId` must **not** be 22207. That number is VeilVM’s HyperSDK app id.

## Privacy

Canonical runtime facts: `specs/VEIL_LOCAL_RUNTIME_STATUS_2026-08-27.md`.

| Surface | v1 status |
|---|---|
| VeilVM commit / reveal / proof / clear | Local path: opaque `VEILENC1` commit, reveal share required, groth16 `shielded-ledger-v1` clear. Circuit SHA256-binds fills + commitment/nullifier/state-root slots. Not in-circuit matching. Not Fuji. |
| Tx gossip | **VTG2 2-of-3** (Shamir + X25519). Encryption-required implies threshold `t>=2`. Outer VTG1 rejected. One key cannot decrypt. Local RPC `SubmitTx` is still plaintext ingest. |
| Companion intent **events** | Commitment + nullifier only (current gateways) |
| Companion ERC-20 / DEX / VAI (out of v1 rails) | Public if deployed; do not ship in v1 |
| Frontend / Polymarket | Public, external catalog. Not VEIL settlement. |

Do not claim full-stack anonymity. Do not call shared-AES gossip a private mempool.

## Companion v1 rails (compile with `FOUNDRY_PROFILE=rails forge build`)

- `contracts/core/WVEIL.sol`
- `contracts/core/VeilFaucet.sol` (testnets only)
- `contracts/bridge/VeilBridgeMinter.sol`
- `contracts/bridge/VeilOrderIntentGateway.sol`
- `contracts/bridge/VeilLiquidityIntentGateway.sol`
- `contracts/identity/ZeroIdVerifier.sol`

Everything else in `veil-contracts` is **parked**: Maker port, Olympus, UniV2, treasury, Keep3r, experimental meme/404. Keep in git. Do not deploy for v1. Do not describe as “the VEIL protocol.” `VeilKeep3r` is not a genesis bucket and is not on the launch path.

Native AMM, VAI mint/burn, COL, fee router live **only** as VeilVM actions 7–14.

## Relayer seam (required for dual-chain)

1. User `submitIntent(commitment, nullifier)` on companion.
2. Envelope bytes stay off-chain. Relayer checks `sha256(envelope) == commitment`.
3. Relayer posts to VeilVM `/evm/intents/execute` or `/evm/liquidity/execute`.
4. Relayer `markIntentExecuted(intentId, veilTxHash)`.

No relayer ⇒ companion and VeilVM are two products, not a stack.
