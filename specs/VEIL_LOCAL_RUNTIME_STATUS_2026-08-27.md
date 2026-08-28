# VEIL local runtime status

Date: 2026-08-27  
Authority: this file for **what is running locally**. Normative whitepaper MUSTs remain in `VEIL_V1_NATIVE_PRIVACY_SPEC.md`. Local ≠ Fuji ≠ mainnet.

## Network

- VeilVM HyperSDK app-id `22207` on local avalanchego 1.13.
- Chain `bdRGUMA7rzZFXjbn1ePTjqhAUfTjW94e69p7qZd4puZ3uEosL`.
- Companion rails on anvil `31337` (not chainId 22207).
- Fuji / mainnet: not this runtime.

## Privacy (what is true)

| Layer | Live | Not live |
|---|---|---|
| Order envelopes `VEILENC1` | Yes. Commit ciphertext, reveal window key, then prove/clear. | Anonymous books after reveal. |
| Tx gossip | **VTG2**, Shamir + X25519, **t=2 n=3**. `txGossipEncryptionRequired` fails closed unless threshold. Outer VTG1 rejected. One committee key cannot open gossip. | Production 13-of-20 / DKG. |
| RPC ingest | Order-router `SubmitTx` is still plaintext to the local block producer. That is how a one-node chain includes txs. | Encrypted RPC submit. |
| Groth16 `shielded-ledger-v1` | Consensus-gated. Preimage binds fills, commitments, nullifiers, prev/next state-root **slots**. | In-circuit matching, merkle inclusion, or balance conservation. Digest-binding only. |

Do not claim a private mempool from shared AES (VTG1) or t=1. Do not claim full-stack anonymity.

## Economy (exercised locally)

Native actions 7–14: fee router 70/20/10, COL tranche release, VAI mint/burn with backing floor, VEIL-VAI AMM add/swap/remove. Router routes: `/native/mint-vai`, `/native/burn-vai`, `/native/route-fees`, `/native/release-col`, `/intents/native/liquidity/execute`.

## Companion rails

Order + liquidity intent gateways, WVEIL, faucet. Relayer: `submitIntent` → mailbox envelope → VeilVM execute → `markIntentExecuted`. Local Teleporter is a mock.

## Proof fixtures

`veilvm/zk-fixture-new` groth16 shielded-ledger VK sha256 `7618a647534c5cc47586f8ad778264a8dfc1a5da71e557db13607bfeae07a5a9`.
