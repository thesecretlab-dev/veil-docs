# VEIL Companion EVM Intent Rails

Companion EVM is the **public on-ramp** to VeilVM. It is not a second copy of the protocol.

v1 layering: [`../architecture/VEIL_STACK.md`](../architecture/VEIL_STACK.md).

## v1 rails (deploy these)

Sources live in `thesecretlab-dev/veil-contracts`. Compile with `forge build --profile rails`.

- `VeilOrderIntentGateway.sol` — commit-only order intent (`commitment`, `nullifier` in events)
- `VeilLiquidityIntentGateway.sol` — commit-only liquidity intent
- `VeilBridgeMinter.sol` — Teleporter minter for WVEIL (placeholders in old address JSON are not production)
- `WVEIL.sol` — wrap companion native VEIL as ERC-20
- `ZeroIdVerifier.sol` — ZER0ID Groth16 verifier
- `VeilFaucet.sol` — testnets only

## Not v1 rails (do not deploy as “the protocol”)

VAI mint/burn, UniV2, COL, fee router, bonds, staking live on **VeilVM** (actions 7–14). Solidity Maker DSS, Olympus, `VeilVAI`, `VeilTreasury`, `VeilUniV2Dex`, Keep3r, meme/404 stay in git as parked ports. Keep3r has **no genesis allocation** and is not on the launch path.

## SDK

See `sdk/README.md` for the TypeScript SDK with:
- `VeilClient` — Intent submission, token queries, DEX operations
- `VeilBridge` — Cross-chain bridge (mint/burn WVEIL)
- React hooks (`VeilProvider`, `useVeil`, `useVeilBalances`, etc.)
- Envelope builder (commitment/nullifier generation)

Purpose:

- users submit signed EVM tx intents from wallet UX
- relayers consume `IntentSubmitted` / `LiquidityIntentSubmitted` events
- relayers forward intents to VeilVM order + liquidity router endpoints
- relayers mark executed intents on EVM after VEIL commit succeeds

## Intent Model

Order and liquidity intent fields:

- `commitment` (`bytes32`) = `sha256(envelope_bytes)`
- `nullifier` (`bytes32`)
- `envelope` (`bytes`) opaque payload (recommended encrypted), delivered to relayer off-chain

Opaque policy:

- EVM contracts store and emit only commitment/nullifier/hash metadata.
- Domain-specific fields (market IDs, side/outcome, amounts, operation details) belong in the envelope and relayer policy, not EVM logs.
- Both gateways use commit-only submit: `submitIntent(bytes32 commitment, bytes32 nullifier)`.
- Envelope bytes are not included in EVM calldata.

## Router Mapping

Forward `IntentSubmitted` event data to:

- `POST /evm/intents/execute`

Router expects:

```json
{
  "intentId": "0x...",
  "envelope": "0x<opaque-bytes>",
  "commitment": "0x<sha256-envelope>",
  "nullifier": "0x<bytes32>",
  "marketKey": "0x... (optional if envelope contains market id)",
  "marketType": "polygon_native (optional)",
  "routingFeeBps": 3,
  "sourceTxHash": "0x..."
}
```

Notes:

- Relayer must verify `sha256(envelope) == commitment` before forwarding.
- Contract events intentionally avoid emitting trader and amount fields.
- Relayer sources envelope bytes from off-chain mailbox/transport keyed by `intentId` or `commitment`.

## Liquidity Router Mapping

Forward `LiquidityIntentSubmitted` event data to:

- `POST /evm/liquidity/execute`

Router expects:

```json
{
  "intentId": "0x...",
  "envelope": "0x<opaque-bytes>",
  "commitment": "0x<sha256-envelope>",
  "nullifier": "0x<bytes32>",
  "operation": "add_liquidity",
  "asset0": 0,
  "asset1": 1,
  "amount0": 1000000,
  "amount1": 2000000,
  "minLP": 1,
  "sourceTxHash": "0x..."
}
```

Liquidity events on EVM now emit only commitment/nullifier/hash metadata. Operation and amount parameters are supplied to the router from relayer envelope handling.

Liquidity operations:

- `create_pool` (`0`): `asset0`, `asset1`, `feeBips`
- `add_liquidity` (`1`): `asset0`, `asset1`, `amount0`, `amount1`, `minLP`
- `remove_liquidity` (`2`): `asset0`, `asset1`, `lpAmount`, `minAmount0`, `minAmount1`
- `swap_exact_in` (`3`): `assetIn`, `assetOut`, `amountIn`, `minAmountOut`

Asset IDs:

- `0 = VEIL`
- `1 = VAI`

Current execution model note:

- VeilVM action actor is the router signer key (`ORDER_ROUTER_PRIVATE_KEY`), not the EVM sender directly.

Headers:

- `x-relay-secret: <ORDER_ROUTER_RELAY_SECRET>`

On success, router returns `VeilTxHash` and marks the intent as accepted for dedupe.

Automated relaying:

- Use `scripts/relay-opaque-intents.mjs` (or `npm run relay:opaque` from `scripts/`) to scan/watch companion gateway events, resolve envelopes from an off-chain mailbox, forward to router, and optionally call `markIntentExecuted`.

## Execution Marking

After successful router execution:

- call `markIntentExecuted(intentId, veilTxHashBytes32)` from the configured relay executor.
- for liquidity intents, call `markIntentExecuted` on `VeilLiquidityIntentGateway`

`VeilTxHash` returned by router is a `0x`-prefixed 32-byte value and can be passed directly as `bytes32`.
