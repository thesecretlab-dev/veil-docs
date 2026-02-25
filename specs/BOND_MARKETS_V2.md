# VEIL Bond Markets v2

## Scope

This document defines the VEIL-native bond and stability surface in the VM.

## Implemented

Actions:

- `BondDeposit` (`19`)
- `BondRedeem` (`20`)
- `CreateBondMarket` (`21`)
- `PurchaseBond` (`22`)
- `RedeemBondNote` (`23`)

Storage primitives:

- `BondState`
- `BondMarket`
- `BondNote`
- `TreasuryLP` balance accounting

RPC:

- `bondstate`
- `bondmarket`
- `bondnote`
- `treasurylp`

## Bond Type Mapping

- `Reserve` (`BondTypeReserve=0`): quote=`VAI`, payout=`VEIL`
- `Inverse` (`BondTypeInverse=1`): quote=`VEIL`, payout=`VAI`
- `VEIL` (`BondTypeVEIL=2`): quote=`VEIL`, payout=`VEIL`
- `Liquidity` (`BondTypeLiquidity=3`): quote=`LP`, payout=`VEIL`

## Price and Payout

Let `priceBips` be the live market price after decay and bounds.

- `Reserve/VEIL/Liquidity`: `payout = quote * 10_000 / priceBips`
- `Inverse`: `payout = quote * priceBips / 10_000`

After each purchase:

- `market.sold += payout`
- `market.startPriceBips = min(maxPriceBips, livePrice + tuneStepBips)`
- `market.lastTuneMs = now`

## Liquidity Reservation

To prevent redemption-time insolvency and race conditions:

- Payout liquidity is reserved at purchase time.
- For `VEIL` payouts, reservation is taken from `TreasuryState.Live`.
- For `VAI` payouts, reservation is taken from `ReserveState.VAIBuffer`.

Redemption only credits the claimant and marks the note claimed.

## Stability Layer

Actions:

- `SetYRFConfig` (`24`)
- `RunYRFWeeklyReset` (`25`)
- `RunYRFDailyBeat` (`26`)
- `SetRBSConfig` (`27`)
- `TickRBS` (`28`)

YRF behavior:

- Weekly reset stages new weekly yield and activates prior staged yield.
- Daily beat consumes budget with backing recycle and writes buyback/burn accounting.

RBS behavior:

- Tick updates moving average and wall/cushion bounds.
- Tick recomputes capacities from reserve depth and applies regen rules.
- Tick toggles active lower/upper cushion market IDs under configured conditions.
