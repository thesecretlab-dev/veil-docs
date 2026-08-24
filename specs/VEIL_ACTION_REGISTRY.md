# VEIL Action Registry

Status: Canonical for GitHub `veilvm`  
Date: 2026-08-24  
Canonical SHA: `9ce05eec1a3bc11df1def10d5a134e4c29803ac3`  
Source of IDs in binary: `veilvm/consts/types.go`  
Runlist item: **A03**

Handshake / ANIMA docs claim **42 actions (IDs 0–41)**. The public VM implements **19 actions (IDs 0–18)**. Until 19–41 exist in this tree and `go test ./...` is green, **public copy must say 19 native actions**, not 42.

## Implemented in `types.go` (binary)

| ID | Const | File | Domain |
|---:|---|---|---|
| 0 | `TransferID` | `actions/transfer.go` | Transfer |
| 1 | `CreateMarketID` | `actions/create_market.go` | Markets |
| 2 | `CommitOrderID` | `actions/commit_order.go` | Markets |
| 3 | `RevealBatchID` | `actions/reveal_batch.go` | Markets |
| 4 | `ClearBatchID` | `actions/clear_batch.go` | Markets |
| 5 | `ResolveMarketID` | `actions/resolve_market.go` | Markets |
| 6 | `DisputeID` | `actions/dispute.go` | Markets |
| 7 | `RouteFeesID` | `actions/route_fees.go` | Fees |
| 8 | `ReleaseCOLTrancheID` | `actions/release_col_tranche.go` | Fees / COL |
| 9 | `MintVAIID` | `actions/mint_vai.go` | Stablecoin |
| 10 | `BurnVAIID` | `actions/burn_vai.go` | Stablecoin |
| 11 | `CreatePoolID` | `actions/create_pool.go` | Liquidity |
| 12 | `AddLiquidityID` | `actions/add_liquidity.go` | Liquidity |
| 13 | `RemoveLiquidityID` | `actions/remove_liquidity.go` | Liquidity |
| 14 | `SwapExactInID` | `actions/swap_exact_in.go` | Liquidity |
| 15 | `UpdateReserveStateID` | `actions/update_reserve_state.go` | Risk |
| 16 | `SetRiskParamsID` | `actions/set_risk_params.go` | Risk |
| 17 | `SubmitBatchProofID` | `actions/submit_batch_proof.go` | ZK |
| 18 | `SetProofConfigID` | `actions/set_proof_config.go` | ZK |

Helpers (not TypeIDs): `glyph.go`, `proof_hash.go`, `proof_verifier.go`, `zk_metrics.go`, `amm_common.go`.

## Spec-only (not in this tree)

From `veil-frontend/docs/ANIMA_ARCHITECTURE.md` and `veil-docs/specs/BOND_MARKETS_V2.md`. Do not advertise as live.

| ID | Name | Spec |
|---:|---|---|
| 19 | BondDeposit | `BOND_MARKETS_V2.md` |
| 20 | BondRedeem | `BOND_MARKETS_V2.md` |
| 21 | CreateBondMarket | `BOND_MARKETS_V2.md` |
| 22 | PurchaseBond | `BOND_MARKETS_V2.md` |
| 23 | RedeemBondNote | `BOND_MARKETS_V2.md` |
| 24 | SetYRFConfig | `BOND_MARKETS_V2.md` |
| 25 | RunYRFWeeklyReset | `BOND_MARKETS_V2.md` |
| 26 | RunYRFDailyBeat | `BOND_MARKETS_V2.md` |
| 27 | SetRBSConfig | `BOND_MARKETS_V2.md` |
| 28 | TickRBS | `BOND_MARKETS_V2.md` |
| 29 | LiquidateCDP | ANIMA architecture |
| 30 | SetVVEILPolicy | ANIMA architecture |
| 31 | StakeVEIL | ANIMA architecture |
| 32 | WrapVVEIL | ANIMA architecture |
| 33 | UnwrapGVEIL | ANIMA architecture |
| 34 | UnstakeVEIL | ANIMA architecture |
| 35 | ClaimOracleReward | ANIMA architecture |
| 36 | SlashOracle | ANIMA architecture |
| 37 | RegisterBloodsworn | ANIMA architecture |
| 38 | UpdateBloodswornScore | ANIMA architecture |
| 39 | SetGlobalPause | ANIMA architecture |
| 40 | SetActionPause | ANIMA architecture |
| 41 | SetRevealCommittee | ANIMA architecture |

## Policy

1. `consts/types.go` is the only count that may appear as “native actions” in UI.
2. Porting 19–41 is Phase D work on this canonical SHA, or a signed reduction of the handshake claim.
3. Josh-host mapping `veil-native-api-v1-mapping.md` is **not recovered** on this PC.
