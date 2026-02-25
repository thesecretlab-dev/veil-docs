# VEIL Mainnet + Chainlink Bridge Runbook

Status: Active  
Date: 2026-02-20  
Scope: Activate and verify live Avalanche mainnet and Chainlink connectivity for VEIL bridge operations

## 1. Goal

Move from placeholder bridge metadata to a live mainnet profile and verify:

1. Avalanche mainnet RPC connectivity.
2. Canonical Teleporter contracts exist on mainnet.
3. Chainlink feeds are live and not stale.
4. A funded mainnet relayer key is available for live bridge message relay.

## 2. Files Added

- `scripts/mainnet-bridge.config.template.json`
- `scripts/check-live-mainnet-bridge.mjs`
- `scripts/activate-mainnet-bridge.mjs`

## 3. Quick Start

From `hypersdk/examples/veilvm/scripts`:

```powershell
Copy-Item .\mainnet-bridge.config.template.json .\mainnet-bridge.config.json -Force
npm.cmd run bridge:activate-mainnet -- --config .\mainnet-bridge.config.json
npm.cmd run bridge:check-mainnet-live -- --config .\mainnet-bridge.config.json
```

## 4. Required Env For Live Relay

For full live bridge relay (not just read connectivity checks), set:

```powershell
$env:MAINNET_RELAYER_PRIVATE_KEY = <REDACTED>
```

The key must hold enough AVAX for destination-chain relay gas.

## 5. Current Known Blocker Pattern

If `bridge:check-mainnet-live` reports:

- `MAINNET_RELAYER_PRIVATE_KEY <REDACTED> required...`
- or relayer balance below minimum

then mainnet reads are live, but cross-chain message relay is not yet executable.

## 6. Verification Output

`bridge:check-mainnet-live` prints JSON with:

- `rpc.chainId` and latest block
- Teleporter code checks (`registry`, `messenger`)
- Chainlink feed round freshness (`updatedAt`, `ageSeconds`, `stale`)
- Relayer key funding status
- `overallPass`

Use this JSON as launch-evidence input for bridge readiness.

## 7. Current Execution Notes (2026-02-20)

- Mainnet connectivity check is green (`overallPass=true`) when `MAINNET_RELAYER_PRIVATE_KEY` is set and funded.
- `bridgeRelayer1` (`0x7deFD0...Ad53`) funded on Avalanche mainnet.
- VEIL-side ICM contracts are live:
  - messenger: `0x253b27...5fcf`
  - registry: `0xa02273...020B`

### Remaining Bridge Blocker

Relayer deploy/start can still fail with:

- `failed to connect to sufficient stake: context deadline exceeded`
- repeated `connectedWeight: 0` for subnet `pirneQjQ...` (VEIL subnet)

Meaning: relay process cannot establish sufficient validator connectivity for the VEIL subnet despite RPC reachability. This is a network/validator reachability issue, not a Chainlink/mainnet RPC issue.
