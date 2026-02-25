# VEIL Native Flywheel Execution Playbook

Date: 2026-02-21  
Status: Active

## 1. Core Launch Position

VEIL launches with:

- native stablecoin: `VAI`
- native treasury reserve controls
- native resource asset: `VEIL`

Primary objective: maximize **treasury NAV accretion** at launch, not short-term token optics.

## 2. Economic Policy (Launch)

1. `POL first`: treasury owns core `VEIL/VAI` LP.
2. Conservative VAI issuance at start (`15-25%` of debt ceiling live in week 1).
3. Grow depth before incentives.
4. Recycle captured fees into treasury LP principal.
5. Open bond markets only after depth and peg stability.
6. Enforce hard stop controls on mint/incentive expansion when risk metrics degrade.

## 3. KPI Triggers (Non-Negotiable)

1. Depth KPI:
   - target: `100 VAI` trade slippage `<= 1.0%`
   - trigger: if `>1.5%` for two checks, inject treasury LP
2. Backing KPI:
   - backing ratio must remain at/above floor
   - trigger: pause net-new mint expansion when floor is threatened
3. Keeper KPI:
   - target `>99%` successful scheduled runs
   - trigger: if `<97%`, add credits and failover keeper set
4. Treasury KPI:
   - target positive net fee capture after incentives by week 4
   - trigger: tighten emissions and cut non-performing lanes

## 4. Day-by-Day Execution Cadence

## T-24h

1. Freeze launch parameters and publish signed manifest.
2. Run evidence gates:
   - `npm.cmd run audit:flywheel`
   - `npm.cmd run audit:economics`
   - `npm.cmd run audit:vm:privacy`
   - `npm.cmd run check:prelaunch`
3. Ensure keeper/relayer runway is funded.
4. Ensure ownership is hardened (no temp admin in production posture).

## T+0 to T+24h

1. Create/confirm core pool and seed POL (`VEIL/VAI`).
2. Mint conservative initial VAI tranche under debt/epoch limits.
3. Add treasury liquidity at target ratio.
4. Start relayer/watchers:
   - `npm.cmd run relay:opaque:watch -- --from-block <N> --poll-ms 5000`
5. Start liquidity watchdog:
   - `npm.cmd run liquidity:deepen` (when depth trigger fires)

## Day 2 to Day 7

1. Keep epoch loop active:
   - `TickRBS`
   - `RouteFees`
   - `RunYRFDailyBeat` / `RunYRFWeeklyReset`
   - `RebaseVVEIL`
2. Recycle realized LP fees into POL.
3. Expand only if KPI gates remain green.
4. Open limited-capacity bond markets only after depth and peg hold.

## 5. EVM Rail Boundary (Must Hold)

EVM is an access rail, not execution truth.

- Allowed on EVM: wallet ingress, bridge/wrapper rails, message transport, intent commit/nullifier.
- Must remain on VeilVM: matching/clearing, privacy proof validity, treasury/risk invariants, canonical economic state transitions.

Privacy requirement:

- EVM stores commitment/nullifier only.
- Opaque envelope executes on VeilVM.
- No plaintext market details on EVM path.

## 6. Consensus and Staking Clarification

1. Chain consensus model: PoS (Avalanche L1/subnet validator model).
2. Consensus staking today: validator stake lane (operationally AVAX in current setup).
3. `StakeVEIL -> vVEIL` is tokenomics staking, separate from consensus stake.

## 7. vVEIL Relationship

- `vVEIL` is not permanently fixed 1:1.
- It is share-index based; 1:1 only when index is 1.0.
- `gVEIL` is non-rebasing wrapper over vVEIL shares.

## 8. Current Parameter Note (75% Reserve Ratio Decision)

For committed USD derived from `25,000 AVAX` at `$8.50` and `RESERVE_RATIO_BIPS=7500`:

- `exogenousReserveInit = 159,375`
- `vaiDebtCeiling = 159,375`
- `vaiEpochMintLimit = 7,968`
- `vaiBufferInit = 7,968`

These values were frozen into `genesis.launchpad.json` for launchpad planning.

## 9. Operational Rule

Do not execute economic actions against ambiguous chain targets.  
Pin one VEIL chain ID as source of truth for flywheel execution and monitoring.
