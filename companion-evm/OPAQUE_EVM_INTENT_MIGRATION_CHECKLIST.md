# Opaque EVM Intent Migration Checklist

Date: 2026-02-20  
Scope: Move companion EVM rails from cleartext intent metadata to commitment/nullifier-only ingress.

## 1. Contract Upgrade

- Deploy new `VeilOrderIntentGateway` (opaque `submitIntent(commitment,nullifier,envelope)`).
- Deploy new `VeilLiquidityIntentGateway` (opaque `submitIntent(commitment,nullifier,envelope)`).
- Verify both contracts on explorer and archive source + constructor args.
- Set `relayExecutor` on both contracts.

## 2. Relayer + Router Cutover

- Upgrade relayer to consume only commitment/nullifier/hash events.
- Enforce pre-forward check: `sha256(envelope) == commitment`.
- Route orders to `POST /evm/intents/execute` with opaque envelope payload.
- Route liquidity to `POST /evm/liquidity/execute` with opaque envelope payload + execution context.
- Confirm nullifier-based dedupe behavior on router.

## 3. Execution Guarantees

- Verify VM receives `CommitOrder` with envelope+commitment.
- Verify successful `veilTxHash` is returned and mapped to EVM intent ID.
- Mark EVM intent executed with `markIntentExecuted(intentId, veilTxHashBytes32)`.

## 4. Privacy Regression Checks

- Explorer tx input contains only commitment/nullifier/envelope bytes for intent submit calls.
- Explorer logs contain no trader/amount/side/outcome/operation amount payloads.
- No relayer path uses legacy cleartext EVM event fields.

## 5. Operational Evidence

- Archive deployment tx hashes for both gateway contracts.
- Archive at least one successful order relay trace and one liquidity relay trace.
- Update runbooks and payload schemas in:
  - `examples/veilvm/companion-evm/README.md`
  - `examples/veilvm/cmd/veilvm-order-router/README.md`
  - `examples/veilvm/VEIL_EVM_INTENT_RELAY_RUNBOOK.md`

## 6. Rollback Plan

- Keep previous gateway addresses archived as read-only historical rails.
- If cutover fails, pause relayer ingestion and re-point to prior gateway set.
- Do not replay nullifiers across versions without explicit mapping rules.
