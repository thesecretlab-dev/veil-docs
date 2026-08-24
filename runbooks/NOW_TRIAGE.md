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
| Persistent companion rails (WVEIL, gateways, faucet) | **No** — anvil is up; addresses file is empty; e2e deploys then throws away |
| Relayer as a standing service against those rails | **No** — one-shot script only |
| Teleporter / `VeilBridgeMinter` wired | **No** |
| Reveal / proof / clear in the product loop | **No** |
| Fuji L1 | **No** |
| `platform-cli` on this Windows box | **No** |
| Fuji operator gas | **Partial** — P ~0.75 AVAX, C ~0.25 AVAX. Not a launch gate. |

---

## NOW (do these before any Fuji L1 talk)

Gate: **local dual-chain is a stack**, or we explicitly waive EVM and ship VeilVM-only (that waiver is not signed). Until then, Fuji is closed.

| ID | Need | Pass |
|---|---|---|
| N1 | Deploy v1 rails on local anvil and **persist** them: WVEIL, order gateway, liquidity gateway, faucet. Owner = companion-admin or documented anvil key. | `companion-evm.addresses.json` has live 31337 addresses; `cast code` non-empty. Teleporter/bridge minter stay **empty** (do not fake). |
| N2 | Standing relayer: intent on gateway → mailbox → `/evm/intents/execute` → `CommitOrder` → `markIntentExecuted`. Same for one liquidity op. Against **N1 addresses**, not a fresh deploy. | Repeatable e2e; gateway state = EXECUTED. |
| N3 | Local stack start does N1+N2 without a human puzzle. | `run-local-stack.ps1` leaves rails + relayer up. |
| N4 | Commit is not a dead end: operator path for reveal + proof + clear on a local market (script is enough; UI later). | Evidence bundle with tx hashes. |
| N5 | Status honesty: C04 is **not** dual-chain PASS; D09 is **FAIL** until Teleporter is real; this file is the “what now.” | Docs match. |

**NOW exit:** N1–N3 PASS. N4 may lag by one working session. N5 must be true immediately.

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
