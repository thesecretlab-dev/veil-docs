# VEIL EVM Rails Behind-The-Scenes Policy

Date: 2026-02-21  
Status: Active policy note

## 1. Goal

Provide EVM wallet accessibility and liquidity connectivity without moving privacy-critical or treasury-critical logic out of VeilVM.

## 2. Source-of-Truth Boundary

- VeilVM is the source of truth for:
  - private execution and proof validity
  - treasury/risk invariants
  - mint/burn and economic control loops
- EVM rails are interoperability and UX access rails only.

## 3. What EVM Rails Are Allowed To Do

1. Wallet access rail: user-facing intent ingress and status UX.
2. Asset rail: bridge/wrapper ingress-egress for VEIL-family and VAI.
3. Messaging rail: cross-chain delivery and acknowledgements.
4. Liquidity rail: entry-exit connectivity for EVM users.
5. Relayer rail: transport commitments/nullifiers plus opaque envelopes.

## 4. What Must Stay On VeilVM

1. Matching/clearing and price discovery.
2. Privacy proof validity and consensus checks.
3. Treasury accounting and release/mint risk controls.
4. Canonical economic state transitions.

## 5. Privacy Requirement (Non-Negotiable)

- EVM path carries commitment/nullifier only.
- Plaintext order or liquidity details must not be committed on EVM.
- Opaque envelope bytes are relayed and executed on VeilVM.

## 6. Behind-The-Scenes UX Model

1. User signs in EVM wallet once.
2. EVM tx records commit/nullifier.
3. Relayer forwards opaque envelope to VeilVM.
4. VeilVM executes privately and finalizes.
5. UI presents a single flow (processing -> finalized), not chain internals by default.

This preserves a seamless user experience while keeping privacy and economic guarantees native to VeilVM.
