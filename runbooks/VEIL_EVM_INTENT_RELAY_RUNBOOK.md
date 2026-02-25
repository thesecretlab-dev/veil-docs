# VEIL EVM Intent Relay Runbook

Status: Active  
Date: 2026-02-20  
Scope: Companion EVM user intent -> VEIL VM order + UniV2 liquidity execution

Note:

- VEIL-only native ingress is now supported and is preferred for privacy-first deployments.

## 1. Goal

Provide a deterministic path for wallet-originated EVM intents to execute as VEIL-native transactions.

Execution model:

1. User submits intent on companion EVM (`VeilOrderIntentGateway` or `VeilLiquidityIntentGateway`).
2. Relayer consumes `IntentSubmitted` or `LiquidityIntentSubmitted`.
3. Relayer resolves opaque envelope off-chain (mailbox/transport) and calls VEIL router endpoint:
   - orders: `POST /evm/intents/execute`
   - liquidity: `POST /evm/liquidity/execute`
4. Router commits the matching VeilVM action.
5. Relayer marks intent executed on EVM with `markIntentExecuted`.

## 2. Components

- EVM order contract: `companion-evm/contracts/VeilOrderIntentGateway.sol`
- EVM liquidity contract: `companion-evm/contracts/VeilLiquidityIntentGateway.sol`
- Router service: `cmd/veilvm-order-router/main.go`
- Router endpoints:
  - `POST /evm/intents/execute`
  - `POST /evm/liquidity/execute`
  - `POST /intents/native/execute`
  - `POST /intents/native/liquidity/execute`
- Relay helper: `scripts/relay-intent-to-router.mjs`
- Liquidity relay helper: `scripts/relay-liquidity-intent-to-router.mjs`
- Opaque relay worker (event scan/watch + execute marking): `scripts/relay-opaque-intents.mjs`
- Companion registry: `scripts/companion-evm.addresses.json`

## 3. Required Config

Router env:

- `ORDER_ROUTER_PRIVATE_KEY=<REDACTED> signer private key>`
- `ORDER_ROUTER_RELAY_SECRET=<shared relay secret>`
- optional for pool creation: `ORDER_ROUTER_GOVERNANCE_PRIVATE_KEY=<REDACTED> VEIL signer private key>`
- optional: `ORDER_NODE_URL`, `ORDER_CHAIN_ID`, `ORDER_MARKET_ID_PREFIX`, `ORDER_AUTO_CREATE_MARKET`
- optional: `ORDER_ROUTER_ENABLE_EVM_INGRESS=false` (disable companion EVM ingress, VEIL-only mode)
- optional: `ORDER_ROUTER_NATIVE_INTENT_SECRET=<secret>` (require `x-native-intent-secret` for native ingress)
- optional: `ORDER_ROUTER_REQUIRE_OPAQUE_ENVELOPE=true|false` (default `true`)
- optional: `ORDER_ROUTER_MIN_OPAQUE_ENVELOPE_BYTES=<N>` (default `96`)

Relayer env:

- `ORDER_ROUTER_RELAY_SECRET=<same shared secret>`
- `ORDER_ROUTER_URL=http://127.0.0.1:9098/evm/intents/execute` (optional if using default)
- `VEIL_INTENT_MAILBOX_PATH=<path to off-chain envelope mailbox JSON>` (default `scripts/intent-mailbox.json`)

## 4. Policy Rules

- Market type routing:
  - VEIL-native market: `routingFeeBps` must be `0`
  - Polygon-native market: `routingFeeBps >= 3` (`0.03%`)
- Router rejects invalid routing policy when `marketType/nativeNetwork` is provided.
- Router endpoints require `x-relay-secret`.
- Router dedupes repeated nullifier requests in memory.
- Liquidity rail currently supports only:
  - `asset=0` (`VEIL`)
  - `asset=1` (`VAI`)
- `create_pool` is governance-gated in VeilVM; relay requires `ORDER_ROUTER_GOVERNANCE_PRIVATE_KEY`.
- Current execution actor is the router signer key (`ORDER_ROUTER_PRIVATE_KEY`) for both order and liquidity rails.

## 5. Payload Mapping

Map event fields from `IntentSubmitted` to router request:

```json
{
  "intentId": "0x...",
  "marketKey": "0x...",
  "envelope": "0x<opaque-bytes>",
  "commitment": "0x<sha256-envelope>",
  "nullifier": "0x<bytes32>",
  "marketType": "polygon_native",
  "routingFeeBps": 3,
  "sourceTxHash": "0x..."
}
```

Notes:

- Relayer must verify `sha256(envelope) == commitment` before forwarding.
- `marketType` accepts `veil_native`/`polygon_native` (also `veil`/`polygon`).
- EVM events intentionally avoid cleartext trader/amount fields.
- `marketKey` is optional in request when the relayer can derive `marketId` from envelope context.
- Commit-only gateway submits (`submitIntent(bytes32,bytes32)`) do not include envelope bytes in calldata; relayer must source envelope from mailbox/transport.
- Router rejects JSON/plaintext envelopes by default (`ENVELOPE_PRIVACY_REJECTED`); include routing/liquidity hints in relay payload/mailbox hint fields rather than inside cleartext envelopes.

Map event fields from `LiquidityIntentSubmitted` to router request:

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

Liquidity operations:

- `create_pool` (`0`): `asset0`, `asset1`, `feeBips`
- `add_liquidity` (`1`): `asset0`, `asset1`, `amount0`, `amount1`, `minLP`
- `remove_liquidity` (`2`): `asset0`, `asset1`, `lpAmount`, `minAmount0`, `minAmount1`
- `swap_exact_in` (`3`): `assetIn`, `assetOut`, `amountIn`, `minAmountOut`

Liquidity privacy note:

- EVM liquidity intents now carry commitment/nullifier envelopes; amounts are supplied to router execution payloads by relayer policy.
- Router resolves liquidity execution context from relay JSON first, with fallback to JSON envelope fields (`liquidity` / `liquidityExecution` / `execution` objects).
- If envelope bytes are encrypted or non-JSON, relayer must include operation/amount fields in relay JSON.

## 6. Quick Verification

Start router:

```bash
ORDER_ROUTER_PRIVATE_KEY=<REDACTED> ORDER_ROUTER_RELAY_SECRET=<secret> go run ./cmd/veilvm-order-router
```

Forward one test intent:

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<secret> npm run relay:intent -- \
  --intent-id 0x<intent-id> \
  --market-key 0x<market-key> \
  --envelope 0x<opaque-bytes> \
  --commitment 0x<sha256-envelope> \
  --nullifier 0x<bytes32> \
  --market-type polygon_native \
  --routing-fee-bps 3 \
  --source-tx-hash 0x<evm-tx-hash>
```

Expected result:

- HTTP `200`
- `accepted=true`
- `veilTxHash` populated

Then call on EVM:

- `markIntentExecuted(intentId, veilTxHashBytes32)`

Forward one liquidity test intent:

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<secret> npm run relay:liquidity-intent -- \
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

Run automated opaque relay worker (orders + liquidity):

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<secret> EVM_RELAY_EXECUTOR_PRIVATE_KEY=<REDACTED> VEIL_INTENT_MAILBOX_PATH=./intent-mailbox.json \
  npm run relay:opaque -- --from-block <START_BLOCK>
```

Watch mode (continuous polling):

```bash
cd scripts
ORDER_ROUTER_RELAY_SECRET=<secret> EVM_RELAY_EXECUTOR_PRIVATE_KEY=<REDACTED> VEIL_INTENT_MAILBOX_PATH=./intent-mailbox.json \
  npm run relay:opaque:watch -- --from-block <START_BLOCK> --poll-ms 5000
```

Dry-run resolve/forward preview (no router POST, no mark):

```bash
cd scripts
VEIL_INTENT_MAILBOX_PATH=./intent-mailbox.json npm run relay:opaque -- --from-block <START_BLOCK> --dry-run --skip-mark
```

Mailbox bootstrapping:

```bash
cd scripts
cp intent-mailbox.template.json intent-mailbox.json
```

Run static privacy surface guardrail:

```bash
cd scripts
npm run check:opaque-intent-privacy
```

Native VEIL-only submit (no EVM dependency):

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

Native VEIL-only liquidity submit:

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

## 7. Launch Evidence Requirements

Archive:

- intent submit tx hash (EVM)
- router execution response (`veilTxHash`)
- VEIL tx success proof (indexer result)
- EVM `markIntentExecuted` tx hash
- updated `scripts/companion-evm.addresses.json` (`orderIntentGateway`, `liquidityIntentGateway` + tx hash log)

Bridge-enabled launch is blocked until this end-to-end path is reproducibly passing.

## 8. Latest Privacy Enforcement Evidence (2026-02-21)

- Node: `http://127.0.0.1:9660`
- Router: `http://127.0.0.1:9098`
- Chain ID: `2vrbfA1TYYTKofezajo4d6tsnuzvBfPJkVxLM85SaBRuUBJ2J1`
- Bundle: `evidence-bundles/20260221-040214-native-privacy-enforcement/bundle.md`
- Pointer: `evidence-bundles/latest-native-privacy-evidence.md`

Validated results:

- Plaintext envelope rejected at VM execution even with router opaque gate disabled (`HTTP 502`, `ORDER_TX_FAILED`, opaque-bytes message).
- Opaque envelope accepted with router opaque gate disabled (`veilTxHash=0xbb2cb6...eb7c`).
- Hardened ingress restored (`opaqueEnvelopeRequired=true`) and plaintext rejected at router edge (`HTTP 400`, `ENVELOPE_PRIVACY_REJECTED`).
- Opaque envelope accepted in hardened mode (`veilTxHash=0x5c644e...cb24`).

## 9. Frontier Failure Notes (2026-02-22)

- Failure: `ENVELOPE_PRIVACY_REJECTED` on replayed historical intents.
  - Symptom: router returns `HTTP 400` with `message: envelope must use encrypted format VENC1`.
  - Cause: legacy mailbox entry does not encode `VENC1` opaque envelope bytes.
  - Action: seed fresh intent with `VENC1` envelope + matching `sha256(envelope)` commitment and replay from that block.

- Failure: `MARKET_PREP_FAILED` on order relay after VENC1 migration.
  - Symptom: router returns `HTTP 502` and `market prep failed` due rejected `CreateMarket`.
  - Cause: auto-create market path (`ORDER_AUTO_CREATE_MARKET=true`) collides with proof-gated market-create policy in current local posture.
  - Action (local test path): use an existing market key, or use a synthetic `liquidity:`-prefixed market key to bypass auto-create in committed-order relay verification.
  - Action (production path): do not rely on synthetic fallback; ensure intended market is pre-created/authorized and evidence-bound.
