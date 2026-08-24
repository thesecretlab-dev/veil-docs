# VEIL total-supply freeze

Date: 2026-08-24  
Runlist item: **A04**  
Decision: **Adopt genesis.json `990,999,000`.**  
Signer: protocol lead (operator session on this host). No separate financing-revision signature exists.

## Canonical number

```
TOTAL_SUPPLY = 990,999,000 VEIL
```

Allocation in `veilvm/genesis.json` (sums to total):

| Bucket | Amount |
|---|---:|
| Circulating / customAllocation (5% float) | 49,549,950 |
| COL vault locked | 900,000,000 |
| COL vault live | 41,449,050 |
| Keep3r | **0** (dropped 2026-08-24; not a bucket) |
| **Total** | **990,999,000** |

## Conflict (rejected until a signed revision)

| Source | Number | Disposition |
|---|---|---|
| `veilvm/genesis.json` | 990,999,000 | **Canonical** |
| `veilvm/VEIL_EXECUTION_PACKAGE.md` | 990,999,000 | aligned |
| `veilvm/VEIL_MASTER_RUNBOOK.md` | 990,999,000 | aligned |
| `veil-docs/specs/VEIL_WHITEPAPER_ALIGNMENT_MATRIX.md` | 990,999,000 | aligned |
| `veil-docs/specs/VEIL_EXECUTION_PACKAGE.md` | **1,191,449,050** | **stale financing copy — do not use** |
| `veil-docs/runbooks/VEIL_MASTER_RUNBOOK.md` | **1,191,449,050** | **stale — do not use** |
| `veil-frontend/public/maiev/economic-coherence/econ-20260220-143818/` | 1,191,449,050 | stale local econ run |

A later seed/presale rewrite to 1,191,449,050 requires a **signed** financing revision **before** any genesis rewrite. Until then, 990,999,000 is frozen.

## Follow-ups (still A04 until docs match)

- Patch `veil-docs/specs/VEIL_EXECUTION_PACKAGE.md` and `veil-docs/runbooks/VEIL_MASTER_RUNBOOK.md` to 990,999,000.
- Do not change `genesis.json` `initialRules.chainID` here (still Primary Network placeholder `11111…LpoYY` — Phase E).
