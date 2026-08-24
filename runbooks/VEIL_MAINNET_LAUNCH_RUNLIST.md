# VEIL Mainnet Launch Runlist

Status: Active  
Date: 2026-08-24  
Owner: Protocol lead  
Decision mode: Hard go / no-go  
Scope: From **today's actual state** to a **mainnet-ready dual-chain VEIL** (VeilVM L1 + companion EVM).

This runlist is the ordered operator path. Older checklists (`VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`, frontend `docs/mainnet-launch-checklist.md`) remain evidence catalogs. **They do not authorize launch.** Feb 2026 `PASS (local)` and `GO FOR PRODUCTION` are **stale**. They do not count as mainnet evidence.

---

## 0. Current posture (2026-08-24)

| Fact | Status |
|---|---|
| Public L1 on Avalanche mainnet or Fuji | **Not live.** Glacier/P-Chain have no VEIL subnet or chain. |
| `https://veil-rpc.thesecretlab.app` | **Down** (HTTP 530). |
| `https://veil.markets` | **Up** — frontend + Polymarket feeds, not native VEIL settlement. |
| Canonical GitHub VM (`thesecretlab-dev/veilvm`) | Actions **0–18** only. Version `0.0.1`. |
| Protocol handshake / local Josh tree | Claims actions **0–41** (42 total). Bond/YRF/RBS specs add **19–28**. |
| Genesis in git | `totalSupply = 990,999,000`. 5% float `49,549,950`. COL locked `900,000,000`. |
| Later financing docs | `1,191,449,050` + $2M seed/presale. **Conflicts with genesis.json.** |
| Feb 22 G0–G12 | All marked PASS (local). **Do not reuse.** |
| Feb 25 companion PoS | Stuck: disconnected bootstrap validator weight 100, connected weight 1, 20% churn cap, ~399 AVAX needed. NodeIDs not on Primary Network now. |
| Hardened owner `0xB9a05A…96af` | **Lost.** Replacement `0x641597…407B` generated. Owner-gated contracts from old key are non-actionable. |
| This workstation (3090 PC) | VEIL clones under `C:\Users\Justin\src\veil`. Local Windows node + anvil + order-router PASS (Phase C). GitHub MCP + Avalanche MCP available. |
| Original operator host | Josh machine: `C:\Users\Josh\hypersdk\examples\veilvm`. |

**Hard rule:** local PASS ≠ Fuji PASS ≠ mainnet PASS. Each network gets a **new** evidence bundle.

---

## How to use this document

- Execute **in order**. A phase does not start until the previous phase's **exit gate** is PASS.
- Every item has: ID, action, pass criteria, evidence path, blocker if fail.
- Mark items `TODO` / `DOING` / `PASS` / `FAIL` / `WAIVED` (waiver needs protocol-lead signature + reason).
- Unsigned rehearsal packets (`allowUnsigned=true`) are automatic **NO-GO**.
- Avalanche MCP is **read-only**. Subnet create / convert-to-L1 / register-validator are signed locally with operator keys. Never paste private keys into chat or git.

Command roots (once recovered):

```text
VEILVM= <clone>/veilvm          # or recovered hypersdk/examples/veilvm
DOCS=   thesecretlab-dev/veil-docs
EVM=    thesecretlab-dev/veil-contracts
FE=     thesecretlab-dev/veil-frontend
EVID=   $VEILVM/evidence-bundles
```

---

## Kill list (do not start mainnet if any is true)

1. Supply numbers still disagree (990,999,000 vs 1,191,449,050).
2. Public GitHub VM action set ≠ the binary that will run on validators.
3. Lost owner still controls any live treasury / VAI / Keep3r / bridge minter.
4. Companion still has a disconnected majority-weight validator (`NodeID-D26…` class failure).
5. Teleporter/bridge addresses are placeholders.
6. Proof path is `clearhash-v1` or verifier is non-strict.
7. Encrypted mempool uses a shared key instead of threshold decrypt across the committee.
8. Frontend claims native private markets as live.
9. Launch packet is unsigned or missing required signers.
10. `check:prelaunch` `overallPass` is not `true` on the **target** network.

---

## Phase A — Recover source of truth

**Exit gate:** one canonical tree, one supply number, one action registry, stale evidence quarantined.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| A01 | Inventory all VEIL trees: GitHub org, Josh `hypersdk/examples/veilvm`, any zip/backup, `veil-internal`. | Written inventory with last-commit SHAs and which tree has actions 19–41. | `evidence-bundles/source-inventory/latest.md` |
| A02 | Pick **one** VM source of truth. Prefer recovered full tree if 42 actions exist and tests pass; otherwise promote GitHub `veilvm` and port missing actions from specs. | Single repo + SHA frozen as `VEILVM_CANONICAL_SHA`. | tagged commit `veilvm-canonical-<date>` |
| A03 | Publish action registry 0–N in `veilvm/consts/types.go` **and** `veil-docs/specs/VEIL_ACTION_REGISTRY.md`. | GitHub matches binary. Handshake “42 actions” either implemented or officially reduced. | spec + consts + `go test ./...` |
| A04 | Freeze **total supply**. Default: genesis.json `990,999,000` unless a signed financing revision is adopted **before** genesis rewrite. | One number in genesis, execution package, alignment matrix, frontend. | `genesis.json` + signed freeze note |
| A05 | Quarantine Feb 2026 evidence. Keep for archaeology. Do not point `latest.txt` at it for launch. | All `latest.txt` pointers marked `STALE-2026-02` or deleted. | `evidence-bundles/STALE.md` |
| A06 | Key census. List every admin/treasury/bridge/prover/governance address. Mark: have key / lost / rotate. | Census complete. Lost `0xB9a05A…` flagged **do-not-deploy-under**. | `evidence-bundles/key-map/latest.json` (no private keys) |
| A07 | Clone stack onto this operator PC (or a dedicated build host): `veilvm`, `veil-docs`, `veil-contracts`, `veil-frontend`, `zeroid`, `anima`, `veildb`, `veil-internal`. | `go build ./...` and `forge build` succeed. | clone log + build logs |

**A exit:** A02, A03, A04, A06, A07 PASS.

---

## Phase B — Operator workstation and toolchain

**Exit gate:** this host can build VM, companion, and sign P-Chain txs.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| B01 | Install Go 1.23.x, Docker, Foundry, Node 20+, Git. | Versions recorded. | `toolchain.txt` |
| B02 | Install Avalanche **platform-cli** (not deprecated avalanche-cli as primary). Keep avalanche-cli only if child-node runbook still needs it, then pin version. | `platform --help` works. | version dump |
| B03 | Generate **new** operator keys (do not reuse lost hardened owner): P-Chain, C-Chain, companion admin, prover authority, launch-packet signer. Store in OS secret store / HSM / `keybox`. | Addresses published; private material never in git. | `key-map` public addresses only |
| B04 | Fund **Fuji** P/C/X for the deploy key (faucet). Do not fund mainnet yet beyond a small C-Chain gas reserve. | Non-zero Fuji P and C balances. | Avalanche MCP `onchain_lookup` on the address |
| B05 | Secrets layout: `veilvm/secrets/` gitignored. Threshold private env files never centralized. | `.gitignore` + `secrets/README.md` present. | git status clean of keys |

**B exit:** B01–B05 PASS.

---

## Phase C — Local dual-runtime revive (not launch)

**Exit gate:** VeilVM + companion EVM healthy **on this host**, with **new** local evidence.

Target topology (from dual-chain readiness package):

- `veilvm-node`, `veilvm-node-secondary`
- `companion-evm-node-a` (`:9650`), `companion-evm-node-b` (`:9652`)
- `sigagg`, `sigagg-proxy`
- VeilVM RPC `http://127.0.0.1:9660`

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| C01 | `docker compose -f docker-compose.local.yml up -d --build` (or recovered compose). | Containers up. `/ext/health` and `/ext/health/readiness` = 200 on 9660. | health JSON |
| C02 | Verifier config at boot: `enabled=true strict=true groth16_vk_set=true required_circuit_id=shielded-ledger-v1`. | Logged. | node log excerpt |
| C03 | Smoke: `node scripts/smoke-local.mjs --chain-id <local>` including commit/reveal/proof/clear. | PASS. | `EVID/local-revive-<ts>/smoke.md` |
| C04 | Dual-chain readiness: `npm run readiness:dualchain`. | DC0, DC1 PASS. DC2 may fail until companion A/B both ready — then re-run. | `dualchain-*/readiness.md` |
| C05 | **Do not** reuse Feb companion L1 / `NodeID-D26…`. If companion was a half-created mainnet L1, treat it as abandoned. New local companion only. | No code path still targets the stuck validator set. | written abandonment note |
| C06 | Re-run `npm run check:prelaunch`. | `overallPass=true` on **this** local profile. | `control-tower/prelaunch-readiness-<ts>.json` |

**C exit:** C01, C02, C03, C06 PASS. DC2 tracked into Phase D if companion still unhealthy.

---

## Phase D — Protocol freeze (code + genesis)

**Exit gate:** genesis, circuits, privacy scope, and claims are frozen and tested.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| D01 | Finalize genesis allocation so buckets sum to `TOTAL_SUPPLY`: circulating, COL locked, COL live. **No Keep3r bucket.** | `scripts/genesis-launchpad.mjs` freeze artifacts. | `launchpad-freeze-<ts>.json` |
| D02 | Encode COL no-drain + `ReleaseCOLTranche` epoch cap in VM tests. | Tests fail if drain attempted. | `go test` COL suite |
| D03 | Fee router 70/20/10 at genesis. | On-chain/VM accounting matches. | tokenomics check |
| D04 | VAI: debt ceiling, epoch mint throttle, exogenous backing floor. vVEIL LTV=0. | Economic coherence audit PASS. | `npm run audit:economics` |
| D05 | Pin circuit `shielded-ledger-v1`. Archive spec hash, VK/PK hashes, vectors. | G3-class bundle **new**, not Feb copy. | `zk-circuit-assurance/latest.txt` |
| D06 | Privacy: encrypted gossip + threshold decrypt required in production profile. Shared-key-only = FAIL. | `npm run audit:vm:privacy` PASS. | `vm-privacy-audit/latest.txt` |
| D07 | Update privacy-scope matrix: VM lane private; companion EVM public unless opaque-intent is actually deployed and verified. | Matrix matches bytecode. | `veil-frontend/docs/privacy-scope-matrix.md` |
| D08 | Frontend no-overclaim: testnet wording,  action count matches registry, no “live private markets.” | Verbiage audit PASS. | `veil-markets-verbiage-audit-<date>.md` |
| D09 | Companion registry: real Teleporter messenger/registry, `VeilBridgeMinter`, intent gateways. Placeholders FAIL. | `npm run check:companion-primitives` and `check:companion-policy` PASS. | `scripts/companion-evm.addresses.json` |
| D10 | Opaque intents: EVM events emit only commitment/nullifier. Relayer envelope off-chain. | Grep + evmbench on gateway sources. | audit excerpt |

**D exit:** D01–D06, D08, D09 PASS.

---

## Phase E — Fuji L1 (first public chain)

**Exit gate:** VeilVM custom L1 exists **on Fuji**, queryable via Avalanche MCP, with ≥2 healthy validators.

Use MCP `build_plan` `operation=create-l1` `network=fuji` `vm=custom` `name=VEIL` `chainId=22207` `tokenSymbol=VEIL` as the command template. MCP does not sign.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| E01 | Build and register custom **VM ID** from canonical SHA. | VM ID recorded. | `vm-id.txt` |
| E02 | `platform subnet create` on Fuji with operator P-Chain key. | Subnet ID returned. | subnet ID |
| E03 | `platform chain create --genesis genesis.json --vm-id $VM_ID --name VEIL`. | Blockchain ID returned. **Do not** leave genesis `initialRules.chainID` as Primary Network placeholder `11111…LpoYY` in production genesis. | chain ID + genesis hash |
| E04 | Deploy / point validator manager (PoA for Fuji is acceptable; mainnet decision in H). | Manager address recorded. | address + tx |
| E05 | `platform subnet convert-to-l1` with initial validator endpoint. | Convert tx accepted. | P-Chain tx ID |
| E06 | Register second validator (child host or second process). Follow child bootstrap runbook for deps; **do not** depend on dead Cloudflare tunnels. Use public Fuji bootstrap or a live tunnel you control. | ≥2 validators, connected stake meets health (not 1%). | MCP `blockchain_lookup_subnet` **found=true** on Fuji |
| E07 | Public Fuji RPC (dedicated hostname, health 200 from the internet). | `curl https://<fuji-rpc>/ext/health/readiness` = 200 off-box. | URL + health JSON |
| E08 | Point ANIMA / frontend testnet RPC at Fuji, not 127.0.0.1. | Config committed. | env + docs |

**E exit:** E03, E05, E06, E07 PASS. Avalanche MCP lookup of subnet ID succeeds on `network=fuji`.

---

## Phase F — Companion EVM + bridge on Fuji

**Exit gate:** WVEIL round-trip VeilVM ↔ companion ↔ C-Chain Fuji (or companion L1 Fuji) with real Teleporter.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| F01 | Stand up companion as a **new** Subnet-EVM L1 on Fuji (or documented sidecar). Do not resurrect Feb mainnet companion. | Dual health 200. | DC2 PASS |
| F02 | Enable NativeMinter, TxAllowList, ContractDeployerAllowList. Admin and enabled address sets **disjoint**. | `check:companion-policy` PASS. | policy report |
| F03 | Deploy WVEIL, Multicall3, CREATE2, bridge minter, intent gateways, VAI (or wVAI with `vaiOrigin=veilvm-bridge`). | Address registry committed with tx hashes. | `companion-evm.addresses.json` |
| F04 | ICTT / Teleporter: home=VeilVM or companion, remote=C-Chain Fuji. MCP `build_plan operation=ictt` for command template. | Bidirectional message test with finalized tx hashes. | `EVID/fuji-bridge/roundtrip.md` |
| F05 | Opaque order relay: IntentSubmitted → `/evm/intents/execute` → `markIntentExecuted`. | End-to-end PASS. | relay logs + VeilTxHash |
| F06 | Liquidity relay: create_pool / add / swap_exact_in for VEIL/VAI. | PASS. | relay logs |
| F07 | **Dropped.** Keep3r is not v1. Cadence is relayer + operator. Do not deploy `VeilKeep3r`. | N/A | this row |
| F08 | Rotate companion owners off temporary EOA to Fuji multisig (can be 2-of-3 test). | Owner(WVEIL, BridgeMinter, intent gateways) = multisig. | `admin-rotation/*` |

**F exit:** F04, F05, F08 PASS.

---

## Phase G — Fuji proving, economics, ANIMA

**Exit gate:** G0–G12 **re-run against Fuji**, plus sustained ZK and flywheel.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| G01 | Proof-gated consensus on Fuji: no-proof cannot clear; malformed rejected; timeout drill; backup takeover. | All four PASS. | `fuji-launch-gate-<ts>/bundle.md` |
| G02 | ZK trial profile: batch 32/64 (96/128 if hardware allows). p95 block ≤6s, missed-proof <0.5% over long run (target 10k-block equivalent or documented waiver). | Summary JSON meets SLOs or explicit waiver. | `zkbench-out-fuji-*/summary.json` |
| G03 | Threshold key ceremony for **Fuji committee** (`VEIL_PRODUCTION_KEY_CEREMONY_RUNBOOK.md`). MinShares approved. Private keys never centralized. | Rollout audit `overallPass=true`. Runtime marker on every validator: `cryptographic threshold tx gossip keying enabled`. | `key-ceremony/ceremony-<ts>-fuji/` |
| G04 | `npm run audit:flywheel` on Fuji addresses. | PASS. | `flywheel-audit/latest.txt` |
| G05 | `npm run audit:economics` after freeze. | PASS. Liquidity deepen if 100 VAI ref trade slippage >1%. | econ + optional liquidity-depth |
| G06 | ANIMA: fail-closed provisioning/validating. Continuity vs strict split. No simulated milestones. G12 bundle on Fuji RPC. | `anima-readiness` PASS. | `anima-readiness/latest.txt` |
| G07 | ZER0ID: circuits compile; verifier deployed on companion; L1/L2 proof verify. | One age≥18 proof verifies on-chain. | tx hash |
| G08 | VEILdb stores init on operator + at least one peer. | `veildb status` shows stores. | status dump |
| G09 | Frontend: Fuji RPC, claim registry, transparency journal points at **new** evidence IDs. Polymarket remains external. Native trade CTA off or clearly testnet. | Deploy preview PASS. | Vercel preview URL |
| G10 | Control tower: `init-launch-signer-policy.ps1` → `check:prelaunch` → `launch:rehearsal` with **signatures**. | `rehearsal-report.overallPass=true`, `signatureCheckPass=true`, `allowUnsigned=false`. | launch-packet.json |

Map to classic gates (must all be **Fuji PASS**, not local):

| Gate | Phase G item |
|---|---|
| G0 chain health | E07 + C/E health |
| G1 proof-gated | G01 |
| G2 native privacy | D06 + G03 |
| G3 full ZK circuit | D05 + G02 |
| G4 tokenomics/COL | D01–D03 |
| G5 VAI risk | D04 |
| G6 bridge | F04–F06 |
| G7 companion policy | F02 |
| G8 audit | evmbench 0 critical/high |
| G9 drills | G01 |
| G10 keys/admin | F08 + G03 |
| G11 rehearsal | G10 |
| G12 ANIMA | G06 |

**G exit:** every classic gate Fuji PASS + G10 packet signed.

---

## Phase H — Mainnet architecture freeze

**Exit gate:** written mainnet topology, funded, keys in HSM/multisig, no leftover Feb L1.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| H01 | Decide validator manager: PoA (permissioned launch) vs native PoS vs ERC-20 PoS. Default recommendation: **PoA for T0**, convert later — avoids Feb churn/weight trap. | Decision signed. | `mainnet-topology.md` |
| H02 | Target validator set size (whitepaper 20, `t=13`). Fuji may be smaller; mainnet T0 minimum **5** with documented path to 20. | Roster of NodeIDs + operators. | `validator-roster.json` |
| H03 | Mainnet AVAX funding plan: P-Chain for subnet/L1 txs, C-Chain gas, validator deposits. If PoS companion is used, size stake so **connected** weight ≥80% with margin. Never start with a single weight-100 bootstrap you cannot churn off. | Balances on deploy key; budget sheet. | MCP lookup + budget |
| H04 | Production key ceremony for **mainnet committee** (new keys, not Fuji keys). | Manifest + custody attestations. | `ceremony-<ts>-prod/` |
| H05 | Production admin: Gnosis/Safe (or equivalent) on C-Chain + companion. Timelock ≥48h on economic params. Emergency pause = pause intake/deploy only, **no drain**. | Owners match census. | Safe address + policy |
| H06 | Confirm no contract still owned by lost `0xB9a05A…`. Redeploy if needed. | Owner grep = Safe only. | ownership dump |
| H07 | DNS: `rpc.veil.markets`, staking endpoint, docs. Do not reuse dead `veil-rpc.thesecretlab.app` without proving origin. | Public health 200. | DNS + curl |
| H08 | Legal/disclosure: testnet → mainnet copy, risk, selective-disclosure policy. | Signed. | disclosure.md |

**H exit:** H01, H04, H05, H06 PASS.

---

## Phase I — Mainnet create (closed set)

**Exit gate:** VeilVM L1 on **mainnet**, MCP `found=true`, closed validator set, RPC public, still **not** public trading.

| ID | Action | Pass criteria | Evidence |
|---|---|---|---|
| I01 | Rehearse commands on a dry-run sheet. Compare to Fuji. | Peer review. | `mainnet-create-sheet.md` |
| I02 | `platform subnet create --network mainnet`. | Subnet ID. | tx |
| I03 | `platform chain create` with **frozen** mainnet genesis (hash = D01 freeze). | Chain ID. MCP `onchain_lookup` / `blockchain_lookup_chain` found. | chain ID |
| I04 | Convert to L1 + initial validators from roster. | Validators current; connected stake healthy. | MCP `blockchain_lookup_subnet` |
| I05 | Companion mainnet L1 or documented C-Chain-only rails. If L1: new subnet, **no** inherited D26 set. | DC-equivalent PASS. | readiness bundle |
| I06 | Deploy production contracts; fill address registry; verify on explorer. | Registry committed. | addresses + snowtrace |
| I07 | Teleporter/ICTT VeilVM ↔ C-Chain **mainnet**. | Round-trip PASS. | `mainnet-bridge-live-check.json` |
| I08 | Threshold gossip enabled on all mainnet validators. | Rollout audit PASS. | `tkroll-<ts>` |
| I09 | Seed COL live tranche + wVEIL/VAI pool under caps (controlled, not public). Slippage KPI ≤1% on 100 VAI ref trade **or** documented deepen. | Metrics. | liquidity-depth |
| I10 | Rebuild `check:prelaunch` + signed rehearsal **against mainnet endpoints**. | overallPass true, signed packet. | launch-packet |

**I exit:** I03, I04, I07, I08, I10 PASS.

---

## Phase J — Public mainnet go-live

**Exit gate:** protocol lead GO. Markets/intents opened in order.

### J1 Final verification (T-24h)

1. Fresh readiness JSON. All G0–G12 **mainnet PASS**.
2. Config hash matches freeze.
3. Bridge, RPC, validator connected-stake snapshot.
4. Frontend copy: mainnet, no overclaim, privacy matrix.
5. Incident channel + rollback runbook from packet staged.
6. Relayer gas ≥7 days. No Keep3r credit check.

### J2 Launch-day sequence

| Step | Open | Notes |
|---|---|---|
| J2.1 | RPC + explorer + transparency | Read-only |
| J2.2 | WVEIL wrap/unwrap | Caps on |
| J2.3 | VAI mint/burn under ceiling | Watch backing floor |
| J2.4 | Native VEIL/VAI AMM on VeilVM | Relayer; no Keep3r |
| J2.5 | Opaque order intents | Native markets |
| J2.6 | Bond / YRF / RBS | Only if D-phase actions are in the canonical VM |
| J2.7 | ANIMA onboarding | Fail-closed; no simulated validators |
| J2.8 | Public announcement | After J2.5 stable ≥1h |

If any critical regression: pause intake (not drain), execute rehearsal rollback.

### J3 T+0 to T+30 (foundation flywheel)

From `VEIL_FOUNDATION_FLYWHEEL_LAUNCH_PLAN.md`:

- T+24h: seed native VEIL/VAI depth; relayer healthy; live dashboard.
- D2–D7: extra pairs, fee recycle, MM onboarding after 48h depth hold.
- D8–D30: cut pure emissions; volume/spread incentives; weekly treasury PnL.
- KPIs: 100 VAI ≤1% slippage; relayer >99%; bridge SLO; net fee capture by week 4.

**J exit:** public GO recorded with packet hash + subnet ID + chain ID.

---

## Phase K — Post-launch hardening (not optional, not “done”)

| ID | Action |
|---|---|
| K01 | Grow validator set toward 20 / t=13. |
| K02 | If T0 was PoA, plan convert-to-PoS without majority-weight trap. |
| K03 | Long-run ZK (10k-block class) if waived at G02. |
| K04 | Production selective-disclosure flow (regulated facts only). |
| K05 | Rotate any remaining operator EOAs. |
| K06 | Child-validator onboarding playbook without Conway credit gate, or funded Conway. |
| K07 | `rpc.veil.markets` CNAME owned in VEIL DNS, not only TSL tunnel. |

---

## Dependency graph (short)

```text
A recover tree/supply/keys
  → B toolchain + Fuji funds
    → C local dual-runtime (new evidence)
      → D genesis/circuit/claims freeze
        → E Fuji VeilVM L1
          → F companion + Teleporter Fuji
            → G Fuji G0–G12 + signed rehearsal
              → H mainnet topology + Safe + ceremony
                → I mainnet create (closed)
                  → J public go-live
                    → K harden
```

---

## Command cheatsheet

Local:

```bash
# VM
go build ./...
docker compose -f docker-compose.local.yml up -d --build
curl -sS http://127.0.0.1:9660/ext/health/readiness

# scripts (from veilvm/scripts)
npm run check:prelaunch
npm run launch:rehearsal
npm run audit:vm:privacy
npm run audit:economics
npm run audit:flywheel
npm run readiness:dualchain
node run-critical-phase-gates.mjs
```

Fuji / mainnet L1 (platform-cli; fill from MCP `build_plan`):

```bash
platform keys generate --name veilOps
platform subnet create --key-name veilOps --network fuji   # then mainnet
platform chain create --subnet-id "$SUBNET_ID" --name VEIL --genesis genesis.json --vm-id "$VM_ID"
platform subnet convert-to-l1 --subnet-id "$SUBNET_ID" --chain-id 22207 --manager "$MANAGER" --validators "$VALIDATOR_ENDPOINT"
platform l1 register-validator --balance "$AVAX" --pop "$NODE_POP" --message "$WARP_MESSAGE"
```

Verify with Avalanche MCP (read-only):

- `onchain_lookup` / `blockchain_lookup_subnet` / `blockchain_lookup_chain` / `blockchain_lookup_validator`
- Expect `found=true` on the **target network** before calling that network live.

---

## Evidence index (create as you go)

```text
evidence-bundles/
  source-inventory/
  STALE.md
  key-map/
  local-revive-*/
  fuji-bridge/
  fuji-launch-gate-*/
  zkbench-out-fuji-*/
  key-ceremony/ceremony-*-fuji/
  key-ceremony/ceremony-*-prod/
  dualchain-readiness/
  launch-rehearsal/
  control-tower/
  mainnet-create-sheet.md
  mainnet-bridge-live-check.json
```

Pointers (`latest.txt`) may only reference **non-stale** runs on the network currently being certified.

---

## Related docs

- `VEIL_MASTER_RUNBOOK.md` — execution philosophy (treat Feb status as historical)
- `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md` — G0–G12 definitions
- `CRITICAL_PHASE_CONTROL_TOWER.md` — signed packet rules
- `VEIL_PRODUCTION_KEY_CEREMONY_RUNBOOK.md`
- `VEIL_CHILD_NODE_BOOTSTRAP_RUNBOOK.md`
- `VEIL_COMPANION_EVM_PRIMITIVES_CHECKLIST.md`
- `VEIL_EVM_INTENT_RELAY_RUNBOOK.md`
- `VEIL_FOUNDATION_FLYWHEEL_LAUNCH_PLAN.md`
- `VEIL_V1_NATIVE_PRIVACY_SPEC.md`
- `VEIL_EXECUTION_PACKAGE.md`
- `specs/BOND_MARKETS_V2.md` — actions 19+
- `specs/VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md` — DC0–DC7; Feb 25 recovery is a **warning**, not a template

---

## First actions on this machine (A01–A07, today)

1. Recover or declare missing: Josh `hypersdk/examples/veilvm` (42-action tree).
2. Clone `thesecretlab-dev/{veilvm,veil-docs,veil-contracts,veil-frontend,zeroid,anima,veildb,veil-internal}`.
3. Freeze supply at **990,999,000** or sign a revision.
4. New operator keys; lost owner never used again.
5. Bring local compose up; produce **new** smoke evidence.
6. Then Fuji L1 — not mainnet.
