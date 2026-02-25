# VEIL Blocker Burndown (2026-02-20)

## Scope

This document tracks active launch blockers and concrete unblocking actions from the latest local readiness pass.

## Blockers Cleared In This Pass

1. `F001` (critical) from flywheel audit is cleared.
   - Previous state: DAI/VAI suite coverage `1/10` with 9 missing contracts.
   - Current state: DAI/VAI suite coverage `10/10`, missing `0`.
   - Evidence:
     - `evidence-bundles/flywheel-audit/flywheel-20260220-145840/flywheel-audit.json`
     - `evidence-bundles/flywheel-audit/flywheel-20260220-145840/flywheel-audit.md`
2. `G9` reliability drill coverage is now locally cleared.
   - Current state: targeted bundle passed backup takeover, malformed-proof rejection, and timeout/missed-proof path.
   - Evidence:
     - `evidence-bundles/20260220-200231-launch-gate-evidence/bundle.json`
     - `evidence-bundles/20260220-200231-launch-gate-evidence/bundle.md`
     - `evidence-bundles/latest-launch-gate-evidence.txt`

## Active Blockers (Remaining)

1. `G3` Full ZK Circuit Scope
   - Status: `IN PROGRESS`
   - Blocker: production circuit assurance package is not fully archived.
2. `G4` Tokenomics + Treasury Locks
   - Status: `IN PROGRESS`
   - Blocker: production freeze/sign-off package not finalized.
3. `G5` VAI Risk Controls
   - Status: `IN PROGRESS`
   - Blocker: full production validation pack pending.
4. `G6` Companion EVM Bridge Readiness
   - Status: `IN PROGRESS`
   - Blocker: VEIL <-> C-Chain round-trip blocked by local subnet validator reachability/quorum.
5. `G10` Key Ceremony + Admin Rotation
   - Status: `FAIL`
   - Blocker: `VAI`, `VeilTreasury`, and `VeilKeep3r` are still owned by temporary admin EOA.
6. `G11` Launch Rehearsal
   - Status: `FAIL`
   - Blocker: full production rehearsal packet not completed.

## New Unblocking Tool Added

- Script: `scripts/rotate-evm-admin-ownership.mjs`
- NPM command: `cd scripts && npm run admin:rotate -- --new-owner <MULTISIG_OR_HSM_EOA>`
- Execute mode: `cd scripts && npm run admin:rotate -- --new-owner <MULTISIG_OR_HSM_EOA> --execute`
- Evidence output:
  - `evidence-bundles/admin-rotation/rotation-*/ownership-rotation.json`
  - `evidence-bundles/admin-rotation/rotation-*/ownership-rotation.md`
  - `evidence-bundles/admin-rotation/latest.txt`

## Immediate Next Blocker Sequence

1. Clear `G10` by running admin ownership rotation in execute mode with final hardened owner address.
2. Re-run `npm run audit:flywheel` to confirm only accepted findings remain.
3. Build final rehearsal packet to close `G11`.
