# VEIL Ecosystem Launch Handoff - Mainnet Recovery

Date: 2026-02-24  
Author: Codex session handoff

## 1) Current Truth (verified now)

- VEILVM runtime is up locally:
  - `veilvm-node` and `veilvm-node-secondary` are running.
  - `http://127.0.0.1:9660/ext/health/readiness` returned `200`.
- Companion EVM is **not live on Avalanche mainnet**.
- Existing deployed companion subnet (`VEIL`) is on **Fuji** only:
  - Sidecar networks: `VEIL -> Fuji`, `VEILPOS -> <none>`.
- Mainnet key used for deployment (`veil-funded`) has zero AVAX on C/P/X:
  - C: `0x580358...503A`
  - P: `P-avax14e3nnuvhv4qgqf98ffn0mafr6pnhz044af80lz`
- Fuji validator set for subnet `pirneQjQHD4zdRf5xq5gnkqbByWMkPoqDBoVGFELaa8ZQXUYi` is currently:
  - `NodeID-7ybs...` weight `20`
  - `NodeID-D26...` weight `20`
  - `NodeID-Kvh...` weight `40`

## 2) What Went Wrong (do not repeat)

1. Mainnet objective was mixed with Fuji operations.
2. Legacy/stale subnet metadata caused environment drift (`VEIL` Fuji sidecar still in active path).
3. Companion health was treated as sync issue while stake-connectivity threshold still blocked bootstrap.
4. Validator weight reduction sequence was started but not completed (old validator remained too heavy).
5. Signature aggregation finalization was run from a container path that tried `localhost:9090`, causing completion failure in that attempt.
6. Launch language in docs/checklists was ahead of hard runtime truth for companion mainnet readiness.

## 3) What To Do

1. Treat VEILVM and companion EVM as separate launch tracks until explicit interop proof is re-verified.
2. Keep one environment target at a time (`mainnet` only for current objective).
3. Use `VEILPOS` (PoS-native config) for new companion deployment path, not old PoA assumptions.
4. Verify funding before any deploy command:
   - `avalanche key list --mainnet --keys veil-funded`
5. Deploy companion to mainnet only after funding is confirmed.
6. Export and verify sidecar contains `Mainnet` network metadata after deploy.
7. Only then start local companion nodes against mainnet subnet IDs.
8. Update launch/devlog status only from live evidence, not intent.

## 4) What NOT To Do

1. Do not run Fuji commands when the stated objective is mainnet.
2. Do not reuse deprecated/legacy VEIL2 references in active launch work.
3. Do not mark companion launch as ready while mainnet key balances are zero.
4. Do not assume "containers up" means subnet ready; always check stake-connected bootstrap and RPC.
5. Do not run partial validator manager flows without confirming signature aggregator reachability path.

## 5) Next Operator Checklist (mainnet-only)

1. Fund mainnet deployment key:
   - `veil-funded` C-chain address: `0x580358...503A`
2. Confirm non-zero AVAX on mainnet (`C`, `P`, and as needed for validator manager tx path).
3. Deploy `VEILPOS` to mainnet.
4. Export mainnet details and record:
   - mainnet subnet ID
   - blockchain ID
   - VM ID
   - validator manager addresses
5. Reconfigure companion local runtime to those mainnet IDs (no Fuji defaults).
6. Bring up companion node A/B and verify:
   - readiness healthy
   - subnet bootstrapped
   - EVM RPC answering on chain endpoint
7. Validate block explorer points to the correct companion chain ID/network.
8. Update canonical launch status docs with evidence links and exact timestamps.

## 6) Canonical Files To Update After Recovery

- `C:\Users\Josh\hypersdk\examples\veilvm\VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
- `C:\Users\Josh\hypersdk\examples\veilvm\LIVE_DEVLOG.md`
- `C:\Users\Josh\Desktop\VEIL_HANDOFF_LATEST.txt`

## 7) Operational Snapshot (at handoff time)

- Running containers:
  - `veilvm-node`
  - `veilvm-node-secondary`
  - `veilvm-rpc-gateway`
  - `veil-blockscout*` stack
  - `sigagg`, `sigagg-proxy`
- Companion EVM node containers were stopped to prevent further Fuji drift during mainnet recovery.

## 8) Concrete Remediation Plan (Mainnet-Only)

1. Hard freeze bad path (15 min)
   - Keep Fuji companion nodes stopped.
   - Make companion config fail-closed unless explicit mainnet variables are set.
   - Remove default Fuji subnet/network fallbacks from active runtime config.

2. Mainnet deployment prep (20 min)
   - Use `VEILPOS` as the only companion deployment target.
   - Verify mainnet deploy key balances before running deploy commands.
   - Required funding target: `0x580358...503A`.

3. Deploy companion to Avalanche mainnet (30-90 min)
   - Deploy `VEILPOS` on mainnet.
   - Export deployment details immediately after confirmation.
   - Required outputs: mainnet `SubnetID`, `BlockchainID`, `VMID`, validator manager addresses.

4. Bring up two companion nodes against the mainnet subnet (1-4 hrs initial sync)
   - Reconfigure local companion runtime using exported mainnet IDs only.
   - Start node A/B and verify:
     - readiness healthy
     - subnet bootstrapped
     - EVM RPC serves current block data

5. Explorer cutover (30-60 min + index sync)
   - Repoint Blockscout to the mainnet companion RPC.
   - Reset only Blockscout index state if needed.
   - Verify explorer head tracks live companion block growth.

6. Evidence + docs closeout (20 min)
   - Archive tx hashes, health outputs, chain IDs, explorer proofs.
   - Update canonical launch docs from evidence only:
     - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
     - `LIVE_DEVLOG.md`
     - `VEIL_HANDOFF_LATEST.txt`

## 9) Definition of Done

- Companion EVM is verifiably deployed on Avalanche mainnet.
- Two local companion validators are bootstrapped and serving RPC.
- Explorer indexes the correct companion mainnet chain.
- Canonical VEIL docs reflect the exact live state with evidence links.
