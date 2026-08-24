# VEIL now-triage

Status: Canonical for what we work on **this week**  
Date: 2026-08-24  
Authority: [`architecture/VEIL_STACK.md`](../architecture/VEIL_STACK.md). This file does not authorize Fuji or mainnet.

**Posture:** local VeilVM runs. Companion EVM is **not** a stack. We are **not** ready to create a Fuji L1. Fuji AVAX in the operator wallet is gas, not a green light.

---

## Product principles (do not violate)

1. **VeilVM is the protocol.** Native VEIL, VAI, AMM, COL, commit/reveal/proof/clear live only as actions 0–18.
2. **Companion EVM is rails.** WVEIL, intent gateways, bridge minter, ZER0ID verifier, test faucet. Not a second VAI/DEX/Keep3r.
3. **No relayer ⇒ two products.** `submitIntent` → off-chain envelope → VeilVM execute → `markIntentExecuted`.
4. **Placeholders fail.** Empty Teleporter, chainId 22207 on companion, abandoned Feb packet, lost owner `0xB9a05AFC…`.
5. **Local ≠ Fuji ≠ mainnet.** A local PASS does not move the runlist forward.
6. **Do not skip the EVM seam** to “go live.” Native UI talking to the router signer is not dual-chain.
7. **Do not claim live private markets** or mempool privacy. D06 gossip is not in the binary.
8. **v1 is 19 actions.** 19–41 stay spec-only. Keep3r is not a bucket and not on the path.

---

## What is actually true

| Piece | True now |
|---|---|
| Local VeilVM node | Yes (`:9660`, shielded-ledger-v1 strict) |
| Native create + `CommitOrder` via router | Yes (local UI / `/orders`) |
| Native AMM / VAI / COL tests | Yes |
| Persistent companion rails (WVEIL, gateways, faucet) | **Yes** on anvil 31337 (`deploy-rails.mjs`) |
| Relayer as a standing service against those rails | **Yes** for e2e; `--watch` on stack start |
| Teleporter / `VeilBridgeMinter` wired | **Local mock only.** Not Fuji ICTT |
| Reveal / proof / clear in the product loop | **No** |
| Fuji L1 | **No** |
| `platform-cli` on this Windows box | **No** |
| Fuji operator gas | **Partial** — P ~0.75 AVAX, C ~0.25 AVAX. Not a launch gate. |

---

## NOW (do these before any Fuji L1 talk)

Gate: **local dual-chain is a stack**, or we explicitly waive EVM and ship VeilVM-only (that waiver is not signed). Until then, Fuji is closed.

| ID | Need | Pass |
|---|---|---|
| N1 | Deploy v1 rails on local anvil and **persist** them. | **PASS** — `deploy-rails.mjs` + `companion-evm.addresses.json`. Teleporter = `LocalTeleporter` mock. |
| N2 | Standing relayer against **N1 addresses**. | **PASS** — e2e EXECUTED on persisted gateways. `relay-opaque-intents.mjs --watch`. |
| N3 | Local stack start does N1+N2. | **PASS** — `run-local-stack.ps1` deploys rails and starts relayer watch. |
| N4 | Reveal / proof / clear operator path. | TODO |
| N5 | Status honesty. | **PASS** — C04/D09 are local-anvil, not Fuji. |

**NOW exit:** N1–N3, N5 PASS. N4 (reveal/proof/clear) still open. Fuji still closed.

---

## NEXT (only after NOW exit)

| ID | Need | Why it waits |
|---|---|---|
| X1 | WSL or a Linux host so `platform-cli` runs | Cannot sign subnet/L1 txs from this Windows `platform-cli` today |
| X2 | Fuji VeilVM L1 (runlist Phase E) | Needs X1 + NOW. P-Chain gas is already there |
| X3 | Fuji companion EVM + real Teleporter + `VeilBridgeMinter` (D09 / Phase F) | Needs X2. Anvil cannot satisfy Teleporter |
| X4 | Point frontend / ANIMA at Fuji RPC, not 127.0.0.1 | Needs X2 |
| X5 | Wallet-signed native orders (not the hardcoded anvil 0xf39… / router signer) | Product, after the seam exists |

---

## NOT NOW (do not pick up)

- Mainnet, public `veil-rpc`, converting anything to L1 on mainnet
- Keep3r, Maker, Olympus, UniV2 Solidity, actions 19–41
- Kalshi
- Encrypted gossip / threshold decrypt (D06) as if it were blocking local rails — it blocks **privacy claims**, not the EVM seam
- Funding more than operator gas
- Treating Polymarket catalog as VEIL settlement

---

## Sequence

```
N1 persist rails
 → N2 standing relayer
 → N3 one-command stack
 → N4 reveal/proof/clear path
then X1 toolchain
then X2 Fuji VeilVM
then X3 Fuji companion + Teleporter
```

Skip a box ⇒ we are acting ready. Do not.
