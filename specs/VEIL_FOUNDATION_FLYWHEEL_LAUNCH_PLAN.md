# VEIL Foundation Flywheel Launch Plan

Generated: 2026-02-20

## 1) Launch Objective

Use the VEIL treasury + VAI + UniV2 + Keep3r loop to create immediate on-chain depth, sustained trading activity, and defensible treasury growth in the first 30 days.

## 2) Flywheel Definition

1. Treasury holds reserve assets and controlled VAI mint authority.
2. Treasury seeds and owns core LP depth (wVEIL/VAI first, then VAI/stable bridge pairs).
3. Traders and relayers route volume through VEIL pools.
4. Fees and spread capture return to foundation-owned positions.
5. Keep3r jobs rebalance liquidity and enforce cadence tasks.
6. Improved depth lowers slippage, increasing volume and further fee capture.

## 3) Hard Prerequisites Before Public Launch

1. Keep3r job wiring enabled for foundation infra contracts (`VeilTreasury`, order gateway, liquidity gateway).
2. End-to-end flywheel runtime probe passes (mint, treasury funding, LP swap path, keeper payout gate).
3. Contract audit artifacts published for active scope, with unresolved findings at zero or explicitly accepted.
4. Treasury and Keep3r ownership moved off temporary EOA into hardened control (multisig/timelock lane).
5. Bridge relayer funded and monitored for VEIL <-> Avalanche C-Chain route.

## 4) 30-Day Execution Plan

### Phase A: T-72h to T-0 (Readiness Lock)

1. Freeze launch parameters.
2. Publish launch parameter manifest:
   - VAI mint ceiling and epoch cap.
   - Minimum LP depth targets per pool.
   - Keep3r bond minimum and payout rates.
3. Run flywheel audit script and ship signed evidence bundle.
4. Run incident drill:
   - Relayer halt.
   - Keeper outage.
   - Oracle stale data path.
5. Approve go/no-go from foundation ops + security owners.

### Phase B: T+0 to T+24h (Liquidity Ignition)

1. Seed primary pool:
   - Pair: `wVEIL/VAI`.
   - Target: enough depth so a `100 VAI` reference trade causes <=1% slippage.
2. Seed bridge-facing pool:
   - Pair: `VAI/AVAX-side stable route` (direct or routed bridge lane).
3. Activate keeper payout lane and verify first successful work cycle on-chain.
4. Run treasury cadence:
   - Controlled VAI mint to LP provisioning wallet.
   - LP add on target ratios.
5. Publish live dashboard with first-hour metrics.

### Phase C: Day 2 to Day 7 (Depth Expansion)

1. Expand LP coverage to additional routed pairs used by intents/routers.
2. Start fee recycling:
   - Route realized LP fees back into LP principal on schedule.
3. Enable liquidity mining only on target pairs with utilization thresholds.
4. Keep3r jobs run every epoch for:
   - Credit top-up checks.
   - LP ratio rebalance.
   - Treasury reserve policy checks.
5. Trigger external market maker onboarding once depth thresholds hold for 48h.

### Phase D: Day 8 to Day 30 (Efficiency + Retention)

1. Reduce pure emissions, increase performance-based rewards.
2. Shift incentives from TVL-only to volume-and-spread quality.
3. Add cross-chain routing incentives where bridge latency and fill quality pass SLO.
4. Begin treasury PnL reporting cadence:
   - Weekly fee capture.
   - Net mint/burn delta.
   - LP inventory risk.
5. Propose governance transition for parameter updates and emergency controls.

## 5) Capital Allocation Framework (Launch Window)

1. 45% foundation launch capital: core LP bootstrapping (`wVEIL/VAI`).
2. 20%: bridge and cross-chain routing depth support.
3. 15%: keeper credits, relayer gas runway, execution reliability.
4. 10%: tactical market-maker and quote-quality incentives.
5. 10%: contingency reserve for volatility and incident response.

## 6) Keep3r Alignment Plan

1. Register and verify required jobs at launch lock.
2. Enforce minimum bonded keeper set with monitored credit runway.
3. Define three job classes:
   - `LiquidityRebalanceJob`
   - `TreasuryCadenceJob`
   - `BridgeHealthJob`
4. Payout policy:
   - Base payout per successful run.
   - Bonus for latency/SLO adherence.
   - Slash conditions for malformed or stale execution.
5. Daily reconciliation between keeper executions and treasury accounting.

## 7) KPI Targets and Triggers

1. Depth KPI:
   - Target: `100 VAI` reference trade slippage <=1% on primary pair.
   - Trigger: if >1.5% for 2 consecutive checks, inject treasury LP.
2. Utilization KPI:
   - Target: daily volume/TVL above baseline threshold.
   - Trigger: if under threshold for 3 days, reallocate incentives to active pairs.
3. Keeper KPI:
   - Target: >99% successful scheduled runs.
   - Trigger: if <97%, increase credits and failover keeper set.
4. Bridge KPI:
   - Target: stable relay confirmations within SLO.
   - Trigger: if SLO breach sustained, reduce cross-chain incentives until stable.
5. Treasury KPI:
   - Target: positive net fee capture after incentives by week 4.
   - Trigger: if negative, tighten emissions and reduce non-performing pairs.

## 8) Audit and Risk Controls

1. Treat flywheel checks as prelaunch gate, not optional monitoring.
2. Require evidence bundle for each major parameter change.
3. Maintain explicit allowlist of active contracts; deprecated scopes excluded by policy.
4. Enforce emergency runbook:
   - Pause mint cadence.
   - Pause incentive distribution.
   - Keep withdrawal and unwind paths available.

## 9) Immediate 24-Hour Tasks

1. Finalize and run flywheel audit bundle.
2. Enable missing Keep3r infra jobs and verify with transaction receipts.
3. Lock launch parameters and publish a single signed manifest.
4. Fund relayer + keeper runway for minimum 7 days.
5. Execute a controlled liquidity ignition rehearsal and capture metrics.
