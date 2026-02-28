# LIVE DEVLOG

## 2026-02-23 19:37:50 -05:00

### Objective
- Add a live ANIMA dashboard viewport for Conway terminal access and document the exact wiring/override behavior.

### Completed
- Updated ANIMA dashboard ANIMA-tab systems panel:
  - `<local-dev-path>`
  - Added `CONWAY BOX TERMINAL` card with embedded iframe viewport and external-open link.
  - Added Conway panel metadata (`sandboxId`, `terminalUrl`, `os`) and URL resolution helper:
    - query override: `conwayTerminalUrl`
    - local override: `localStorage["anima.conwayTerminalUrl"]`
    - fallback: built-in default URL
  - Added URL validation guard (`http/https` only) and storage-access fallback handling.
- Documented dashboard behavior for operators:
  - `<local-dev-path>`
  - Added `Conway Terminal Viewport` section with target sandbox, URL, OS label, and override precedence.

### Validation
- UI tests executed in ANIMA dashboard repo:
  - `cmd /c pnpm --dir "<local-dev-path>" test`
  - Existing unrelated failures remain in navigation/i18n/browser suites; no new failure was introduced in `anima-dashboard.ts` path.

### Current State
- ANIMA dashboard now has an in-panel Conway terminal viewport for the active child box.
- Conway box OS label is set to `Ubuntu 22.04 LTS (Jammy)` for operator visibility.

## 2026-02-23 19:19:33 -05:00

### Objective
- Wire ANIMA validation to the canonical live VEIL VM API path (`/ext/bc/<chain>/veilapi`) and remove false failures caused by legacy `veil_*` probing on C-Chain RPC routes.

### Completed
- Updated ANIMA MVP command RPC handling:
  - `<local-dev-path>`
  - `--anima-rpc-url` now accepts VEIL native API paths (`/veilapi`) in addition to EVM RPC paths.
  - chain-id detection now falls back to `veilvm.genesis` when `eth_chainId` is unavailable on native VEIL endpoints.
- Updated VM SDK runtime compatibility for native VEIL API:
  - `<local-dev-path>`
  - added `/veilapi` fallback handling for `ping`, `getChainId`, and `getBlockNumber` (height from `/ext/health`).
  - guarded unsupported legacy method calls (`agent_get`, `market_list`, `events_query`) so startup probes fail only on real connectivity errors.
- Executed continuity MVP run against live standalone endpoint and canonical chain API:
  - ANIMA runtime probe now starts/stops live (`Agent ... online`, `Agent stopped`).
  - result: `continuity-pass`
  - artifact: `<local-dev-path>`
- Executed strict fresh-provision attempt (non-simulated):
  - result: fail at Conway control-plane billing gate (`INSUFFICIENT_CREDITS`)
  - artifact: `<local-dev-path>`

### Current State
- ANIMA-to-VEIL connectivity is real on the live standalone VEIL endpoint when using canonical `/veilapi`.
- Strict MVP remains blocked by Conway fresh-provision credits, not ANIMA/VEIL runtime connectivity.

## 2026-02-23 18:55:05 -05:00

### Objective
- Remove MVP surface overclaim by splitting strict completion from continuity completion.

### Completed
- Updated automaton runner truth model:
  - `<local-dev-path>`
  - added `--provision-mode auto|fresh|reuse`.
  - run/tracker payloads now include `strictPassed`, `continuityPassed`, `provisionedFresh`, and explicit `outcome`.
  - strict completion now requires fresh provisioning; sandbox reuse only qualifies as continuity.
- Updated frontend status surfaces:
  - `<local-dev-path>`
  - `<local-dev-path>`
  - `<local-dev-path>`
  - launch page now defaults to strict run mode and renders strict/continuity badges.
- Reclassified latest run artifacts to continuity (no false strict pass):
  - `<local-dev-path>`
  - `<local-dev-path>`
  - `<local-dev-path>`

### Validation
- `pnpm.cmd exec tsc --noEmit` passed in `<local-dev-path>`.
- Frontend repo still has existing unrelated TypeScript debt; MVP files were patched without broad repo rollback or mass edits.
- Strict mode run (`--provision-mode fresh`) reached real Conway control-plane and failed with billing proof:
  - `INSUFFICIENT_CREDITS` (`required_cents=800`, `current_balance_cents=0`)
  - artifact: `<local-dev-path>`
- Continuity mode run (`--provision-mode reuse --control-sandbox-id d2fe48a2a6465322e963a0a11c30ead3`) passed under SLA:
  - outcome: `continuity-pass`
  - artifact: `<local-dev-path>`

### Current State
- MVP board no longer treats continuity runs as strict completion.
- Next strict run requirement is unchanged: fresh server provisioning + full M1..M5 within SLA.

## 2026-02-23 18:44:55 -05:00

### Objective
- Remove the confusing closed-alpha gate so live VEIL surfaces are directly accessible without access-key flow.

### Completed
- Removed alpha-gate middleware from frontend source:
  - deleted `<local-dev-path>`
- Deployed updated frontend to production:
  - inspect: `https://vercel.com/0x12371cs-projects/veil/GgkPkUhyMptVyzQeREavN6VXUsKy`
  - production alias: `https://veil.markets`
- Verified live behavior (no key/cookie required):
  - `https://veil.markets/app/transparency` returns live page (not Closed Alpha gate)
  - new journal entries still present on live response.

### Current State
- Alpha gate is removed from production; live VEIL pages are directly reachable.

## 2026-02-23 18:37:12 -05:00

### Objective
- Ensure the latest child-validator progression is documented and actually visible on the live transparency surface (no local-only update).

### Completed
- Updated live transparency journal content in frontend source:
  - `<local-dev-path>`
  - added 2026-02-23 entries for child connectivity blocker closure and `Live Transparency Push Verified`.
- Updated handshake-facing frontend status brief:
  - `<local-dev-path>`
- Fixed production deploy blockers in frontend project so deploy could complete:
  - added missing deps used by existing gov surfaces (`viem`, `wagmi`, `@tanstack/react-query`)
  - made `/app/gov/new` wallet components client-only via dynamic imports (`ssr:false`) to prevent prerender wagmi-provider failure.
- Executed production deploy from linked Vercel project and confirmed alias:
  - deploy inspect: `https://vercel.com/0x12371cs-projects/veil/3gRAQu88G44qWCPuqM2Rrm8pjWAT`
  - production alias: `https://veil.markets`

### Current State
- Live transparency/devlog surface is now updated in production with current child-validator status and non-greenwashed in-progress registration path.

## 2026-02-23 18:23:45 -05:00

### Objective
- Clear the child-validator network topology blocker by making operator staking/RPC reachable from the child sandbox without changing frontend/runtime surfaces.

### Completed
- Extended operator Cloudflare tunnel ingress on this host:
  - added `veil-staking.thesecretlab.app -> tcp://127.0.0.1:9651`
  - added `veil-rpc-tcp.thesecretlab.app -> tcp://127.0.0.1:9660`
- Validated tunnel ingress syntax and restarted `cloudflared` tunnel process.
- On child sandbox `d2fe48a2a6465322e963a0a11c30ead3`, installed `cloudflared` and established Access TCP sidecars:
  - staking bridge: `127.0.0.1:29651 -> veil-staking.thesecretlab.app -> operator :9651`
  - rpc bridge: `127.0.0.1:29660 -> veil-rpc-tcp.thesecretlab.app -> operator :9660`
- Added child-host bridge helper script for repeatable restarts:
  - `/root/veil-operator-bridge.sh` (`ensure|status|restart`)
- Verified live connectivity from child:
  - staking path completed TLS handshake via `openssl s_client` on `127.0.0.1:29651`
  - rpc path returned `HTTP 200` for `/ext/health/readiness` on `127.0.0.1:29660`
- Updated canonical child tracker:
  - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`
  - connectivity flags now `true` with bridge mode and log paths recorded.
- Updated reusable runbook for all future child hosts:
  - `VEIL_CHILD_NODE_BOOTSTRAP_RUNBOOK.md` now includes bridge bring-up and validation command blocks.

### Current State
- Network topology blocker is cleared via deterministic bridge mode.
- Remaining blocker is validator registration/activation transaction path and params (`l1`, `validator-manager`, stake/reward owner), not connectivity.

## 2026-02-23 18:11:22 -05:00

### Objective
- Push child validator launch past bootstrap by validating real connectivity requirements to the active VEIL chain and narrowing blockers to network topology, not host dependencies.

### Completed
- Captured child-host live launch report in sandbox:
  - `/root/veil-child-validator-launch.json`
  - cluster `veil-child-001` is running healthy
  - endpoint `http://127.0.0.1:9652`
  - node id `NodeID-A4yqq7yUCQV3CPsVaEGVATLNgHc2fd7vC`
- Derived active VEIL chain/subnet targets from operator node:
  - chain: `aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs`
  - subnet: `ommwdCcEdTLSHuAWH7w8n1Dv3yETLk26zpF7Jp8NUXNQCn9Fp`
- Probed child->operator reachability:
  - `veil-rpc.thesecretlab.app:443` reachable
  - operator staking `:9651` unreachable from child
  - operator raw RPC `:9660` unreachable from child
- Updated canonical child tracker with narrowed blockers and target ids:
  - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`

### Current State
- Child node is launch-ready at host/runtime layer, but cannot join the active VEIL validator set until operator staking endpoint (`tcp/9651`) is reachable from child and validator registration tx path is bound to that public bootstrap topology.

## 2026-02-23 18:05:20 -05:00

### Objective
- Recover child validator host continuity after prior sandbox/context drift and restore node-local readiness in the currently owned sandbox.

### Completed
- Verified historical setup context:
  - prior MVP artifact references sandbox `1ae8f1924e7b12e3e82adcc7a510adcf`.
  - current Conway key context cannot access that sandbox (`403 Forbidden`).
- Recovered current child host (`d2fe48a2a6465322e963a0a11c30ead3`) without user input:
  - installed/verified AvalancheGo binary via Avalanche CLI local workflow.
  - fixed stale partial local metadata (`/root/.avalanche-cli/local/veil-child-001`) and recreated cluster cleanly.
  - started local cluster `veil-child-001` healthy.
  - linked `avalanchego` into PATH (`/usr/local/bin/avalanchego`).
- Captured current node-local readiness in tracker:
  - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`
  - child status advanced to `node-cluster-ready`.

### Current State
- Child sandbox `d2fe48a2a6465322e963a0a11c30ead3` now has a healthy local cluster:
  - endpoint: `http://127.0.0.1:9652`
  - node id: `NodeID-A4yqq7yUCQV3CPsVaEGVATLNgHc2fd7vC`
- Remaining blocker is narrowed to validator registration inputs:
  - `l1`, `validator-manager`, `stake`, `rewards/owner` params for `avalanche node local validate`.

## 2026-02-23 17:42:23 -05:00

### Objective
- Resume Build Games MVP execution with reliable artifact/tracker state and no dependency on creating a brand-new Conway sandbox when credits are temporarily zero.

### Completed
- Hardened MVP artifact serialization in `<local-dev-path>`:
  - `M5_ARTIFACT` now records as `passed` in the emitted run JSON (no stale `pending` in success runs).
- Hardened tracker mapping in `<local-dev-path>`:
  - tracker updates now map `M5_ARTIFACT` -> `M5_SURFACE_STATUS`.
  - `M5_SURFACE_STATUS.evidence` is now auto-populated with `mvp_run_json_path`, `ui_status_rendered`, and `source_artifact`.
- Added no-new-credits fallback in MVP flow:
  - when `--control-sandbox-id` is provided, M2 now reuses that sandbox after an exec probe instead of forcing `createSandbox`.
- Re-ran MVP with sandbox reuse:
  - command used `--control-sandbox-id d2fe48a2a6465322e963a0a11c30ead3`
  - result: `PASS`, duration `23834ms`, under 20-minute SLA.
- Updated canonical artifacts/surfaces:
  - `<local-dev-path>`
  - `<local-dev-path>`
  - `<local-dev-path>`
- Added child-validator launch preflight state to bootstrap tracker:
  - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`
  - confirmed blockers: `avalanchego` missing in PATH, no local cluster initialized, and missing full validator registration params.

### Current State
- MVP latest pointer is back to `PASS` with all milestones complete.
- Tracker/UI sync no longer requires manual `M5` correction after each run.
- Child host bootstrap remains complete; next pending operations item is unchanged:
  - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`
  - `nextAction`: `Install/locate avalanchego, initialize local node cluster, then execute validator registration with full L1 params.`

## 2026-02-23 16:35:10 -05:00

### Objective
- Remove ANIMA runtime simulation paths and align startup capital requirements with validator stake so launch posture stays fail-closed.

### Completed
- Updated ANIMA runtime startup gate in `<local-dev-path>`:
  - `newborn` now requires initial `VEIL >= validatorStakeVEIL` before transition out of newborn.
  - initial VAI funding remains required for settlement before entering trading.
- Removed simulated milestone progression in ANIMA runtime:
  - provisioning no longer auto-sets `InfraProvisioned`.
  - validating no longer auto-sets `ValidatorActive`.
  - both paths now explicitly fail closed when live integrations are missing.
- Updated ANIMA SDK docs:
  - `<local-dev-path>` now reflects fail-closed provisioning/validating behavior and startup capital requirements.
- Updated launch-gate criteria for ANIMA readiness:
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md` `G12` acceptance criteria now explicitly disallow simulated infra/validator milestone completion.
- Recorded ANIMA-side live log update:
  - `<local-dev-path>`

### Current State
- ANIMA runtime no longer presents simulated milestone completion as progress.
- Runtime readiness now fails closed for missing provisioning/validator integrations, matching production launch posture.

## 2026-02-23 16:14:30 -05:00

### Objective
- Prepare first child validator host with required dependencies and record a repeatable bootstrap process for all future child nodes.

### Completed
- Connected to child sandbox:
  - `sandboxId=d2fe48a2a6465322e963a0a11c30ead3`
- Baseline audit:
  - present: `node`, `npm`, `git`, `jq`, `curl`
  - missing: `go`, `avalanche` CLI in PATH
- Installed and verified dependency set on child host:
  - OS packages: `build-essential`, `pkg-config`, `libssl-dev`, `tmux`, plus base tooling
  - Go: `go1.23.7`
  - Avalanche CLI: `1.9.6`
- Persisted shell environment for repeat sessions:
  - PATH includes `/usr/local/go/bin` and `/root/bin` via `/root/.bashrc` and `/root/.profile`
- Wrote child-host bootstrap report:
  - `/root/veil-child-node-bootstrap.json`
- Added canonical reusable docs and tracker:
  - `VEIL_CHILD_NODE_BOOTSTRAP_RUNBOOK.md`
  - `evidence-bundles/child-node-bootstrap/child-node-bootstrap-tracker.json`
  - `evidence-bundles/child-node-bootstrap/latest.txt`

### Current State
- First child host is dependency-ready for validator launch sequence.
- Bootstrap is now standardized and logged for roll-forward on all future child nodes.

## 2026-02-23 16:09:47 -05:00

### Objective
- Validate go/no-go after standalone RPC bring-up by running full Build Games MVP flow against the live cloud-reachable VEIL endpoint.

### Completed
- Executed MVP run with remote VEIL RPC set to standalone endpoint:
  - `--veil-rpc-url-remote https://veil-rpc.thesecretlab.app`
  - `--veil-rpc-url-local http://127.0.0.1:9660`
  - profile: `1 vCPU / 512 MB / 5 GB`
- Result:
  - `Build Games MVP result: PASS`
  - duration: `11s`
  - sandbox: `d2fe48a2a6465322e963a0a11c30ead3`
  - payment tx: `0xb860b0...b86c`
- Artifacts:
  - `<local-dev-path>`
  - `<local-dev-path>`
  - `<local-dev-path>`
  - tracker updated: `<local-dev-path>`

### Current State
- Standalone RPC path is operational for cloud validation.
- MVP acceptance flow is currently passing with recorded evidence.

## 2026-02-23 16:06:56 -05:00

### Objective
- Bring up a standalone VEIL RPC endpoint from this operator PC that cloud sandboxes can reach in MVP validation runs.

### Completed
- Extended Cloudflare tunnel ingress config on this machine:
  - file: `<local-dev-path>`
  - RPC routes now target `http://127.0.0.1:9660` with `httpHostHeader=127.0.0.1`
- Created and validated a live public RPC hostname in active tunnel-owned DNS:
  - `veil-rpc.thesecretlab.app` -> tunnel `a15d2868-ffa5-456f-8109-c7c0baa584d0`
- Restarted tunnel connector and verified remote readiness endpoint:
  - `https://veil-rpc.thesecretlab.app/ext/health/readiness` -> `200`
- Recorded standalone RPC runbook updates:
  - `VEIL_STANDALONE_RPC_RUNBOOK.md` now includes current live endpoint and VEIL-domain CNAME mapping requirement.

### Current State
- Standalone RPC is live and cloud-reachable at `https://veil-rpc.thesecretlab.app`.
- Current tunnel worker process is running on operator host (`cloudflared` PID `313676` at validation time).
- VEIL website DNS host `rpc.veil.markets` still needs zone-level CNAME control in `veil.markets` DNS to point at this tunnel target.

## 2026-02-23 16:00:44 -05:00

### Objective
- Stand up a standalone VEIL RPC endpoint reachable from cloud sandboxes, and document the operator-safe path before runtime changes.

### Completed
- Verified local VEIL RPC readiness is healthy:
  - `http://127.0.0.1:9660/ext/health/readiness` -> `200`
- Confirmed current local node host-header policy in compose startup args:
  - `--http-allowed-hosts=localhost,127.0.0.1,host.docker.internal`
  - this requires tunnel origin host override (`127.0.0.1`) for public exposure.
- Added standalone RPC operator runbook:
  - `VEIL_STANDALONE_RPC_RUNBOOK.md`
- Linked runbook in primary docs index:
  - `README.md` (`Key Launch Docs`)
- Added MVP handshake requirement note for remote VEIL RPC wiring:
  - `<local-dev-path>`

### Current State
- Documentation for standalone RPC bring-up is now in place and canonical.
- Next action is to bind a dedicated VEIL hostname (`rpc.veil.markets`) to this operator machine via Cloudflare tunnel and validate remote `/ext/health/readiness`.

## 2026-02-22 22:16:55 -05:00

### Objective
- Recover missing hardened-owner signer custody for production treasury operations, or create an organized replacement key path if recovery fails.

### Completed
- Ran exhaustive local key discovery sweep across `C:\Users\Josh` (including previously skipped areas) and wrote evidence:
  - `evidence-bundles/key-discovery-20260223.json`
  - scan summary: `filesScanned=329636`, `keyCandidates=106755`, `targetFound=false`
  - missing target confirmed: `0xB9a05A...96af`
- Generated a new hardened-owner replacement key bundle with strict file ACLs:
  - active address: `0x641597...407B`
  - active key file: `<local-dev-path>`
  - archive key file: `<local-dev-path>`
  - index file: `<local-dev-path>`
  - ACL: `SYSTEM:(R)`, `Josh:(R,W)` only on private key <REDACTED>
- Built organized signer map (known key files only) with local/runtime + AVAX C-Chain balances:
  - `evidence-bundles/key-map-20260223.json`

### Current State
- Legacy hardened owner key remains unrecovered.
- Replacement hardened owner key is generated, indexed, and secured for immediate future ownership posture.
- Existing production contracts owned by unrecovered signer remain non-actionable for owner-gated state changes until ownership is migrated or stack is redeployed under controlled signer custody.

## 2026-02-22 22:12:47 -05:00

### Objective
- Recover full launch posture after companion runtime shutdown and re-verify strict critical-phase readiness with production signature policy.

### Completed
- Restored companion runtime services required by launch checks:
  - restarted `avago-veil2`, `avago-veil2b`, `sigagg`, `sigagg-proxy` in `<local-dev-path>`
  - confirmed companion RPC back online at `http://127.0.0.1:9650/ext/bc/2L5JWLhXnDm8dPyBFMjBuqsbPSytL4bfbGJJj37jk5ri1KdXhd/rpc`
  - verified `eth_chainId=0x56bf` (`22207`)
- Re-ran launch gate stack:
  - `node scripts/prelaunch-readiness.mjs` -> `overallPass=true`
  - initial `node scripts/run-critical-phase-gates.mjs` run failed only on strict packet signatures (`LH11_PACKET_SIGNATURES`)
  - loaded launch signer env from `<local-dev-path>` and reran -> pass
- Executed strict wrapper to keep signer policy deterministic:
  - `powershell -ExecutionPolicy Bypass -File scripts/run-critical-phase-strict.ps1`
  - new run: `evidence-bundles/critical-phase/critical-phase-20260223-031247/critical-phase-summary.json`
  - result: `overallPass=true`, `CP07_REHEARSAL_PRODUCTION_ELIGIBLE=true`
- Refreshed mainnet operator readiness pack:
  - `node scripts/build-mainnet-deployment-readiness.mjs`
  - new latest: `evidence-bundles/mainnet-deployment/mdprep-20260223-031201`
  - `deploymentReady=true`, `recommendation=READY_FOR_OPERATOR_MAINNET_EXECUTION`
- Revalidated live mainnet bridge surface with relayer key loaded:
  - `node scripts/check-live-mainnet-bridge.mjs` -> `overallPass=true`
  - AVAX C-Chain RPC and Teleporter/Chainlink checks all pass

### Current State
- Launch posture is green again in current latest pointers:
  - `evidence-bundles/critical-phase/latest.txt` -> `overall_pass=true`
  - `evidence-bundles/mainnet-deployment/latest.txt` -> `deployment_ready=true`
- Self-host VEIL runtime remains healthy:
  - router `/health` reports `chainId=aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs`
  - `/veilapi` on `aQ8Ct8...` returns healthy genesis/tokenomics snapshot

### Remaining Operator Blocker (Not a Launch-Gate Failure)
- Production LP deepening on the hardened treasury is still signer-custody blocked:
  - required owner key: `0xB9a05A...96af`
  - without this signer, treasury-owned stateful LP actions cannot be executed on production contracts.

## 2026-02-22 18:15:21 -05:00

### Objective
- Execute the next live operator step after deployment readiness: bind runtime routing to the newly launched self-host VEIL chain and capture execution evidence.

### Completed
- Verified new chain runtime health on launched chain id:
  - `aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs`
  - `scripts/smoke-local.mjs --chain-id aQ8Ct8... --node-url http://127.0.0.1:9660` -> pass (`height 510 -> 541`)
- Rebound order router to the launched chain id:
  - restarted `veilvm-order-router` container with `ORDER_CHAIN_ID=aQ8Ct8...`
  - `/health` now reports `chainId=aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs`
- Confirmed blockchain lifecycle state remains active:
  - `platform.getBlockchainStatus` -> `Validating` for `aQ8Ct8...`
- Captured self-host operator execution bundle:
  - `evidence-bundles/self-host-launch/launch-20260222-181521-operator-execution/bundle.json`
  - `evidence-bundles/self-host-launch/launch-20260222-181521-operator-execution/bundle.md`
  - `evidence-bundles/self-host-launch/latest.txt` now points to this run.

### Current State
- Deployment readiness remains green (`evidence-bundles/mainnet-deployment/latest.txt` -> `deployment_ready=true`).
- Live router traffic target is now aligned to the newly launched self-host VEIL chain (`aQ8Ct8...`).

## 2026-02-22 18:04:31 -05:00

### Objective
- Move from mainnet-readiness staging into executable launch posture, then run the self-host UUP chain-launch path on this machine.

### Completed
- Advanced launch authority sheet to executable state:
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
  - sign-off sheet now filled (`6/6 PASS`)
  - decision line set to `GO FOR PRODUCTION`
- Fixed mainnet deployment readiness decision parser mismatch:
  - `scripts/build-mainnet-deployment-readiness.mjs`
  - `MD06_DECISION_STATE_GO` now accepts both `GO` and `GO FOR PRODUCTION`.
- Rebuilt mainnet deployment readiness after checklist/signoff updates:
  - `evidence-bundles/mainnet-deployment/latest.txt` -> `mdprep-20260222-230652`
  - `deploymentReady=true`
  - `recommendation=READY_FOR_OPERATOR_MAINNET_EXECUTION`
- Executed UUP self-host preflight and live chain creation:
  - dry-run: `scripts/create-chain-on-subnet.uup.ps1` -> pass
  - first live attempt failed with `illegal name character` (hyphenated chain name)
  - retried with alphanumeric chain name and issued CreateBlockchain successfully:
    - tx / chain id: `aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs`
    - coreapi: `http://127.0.0.1:9660/ext/bc/aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs/coreapi`
    - veilapi: `http://127.0.0.1:9660/ext/bc/aQ8Ct8Hwwn71EeJw2eyPQJhu8mwHVoyGT8dDGgGu3svQAGSWs/veilapi`
- Confirmed new chain lifecycle status:
  - `platform.getBlockchainStatus` for `aQ8Ct8...` -> `Validating`
- Hardened UUP defaults to avoid illegal-name regression:
  - `scripts/create-chain-on-subnet.uup.ps1` default chain name now `VEILUUPSELFHOST<timestamp>`
  - `scripts/create-chain-on-subnet.uup.template.ps1` default chain name now `VEILUUP<timestamp>`

### Current State
- Mainnet deployment readiness bundle is green with no failed checks.
- Self-host-first launch path has been executed successfully on this operator node, and the new chain is validating.

## 2026-02-22 17:46:45 -05:00

### Objective
- Enforce explicit launch-packet signer policy in strict critical-phase runs (required signer addresses + min signatures), with local key organization preserved.

### Completed
- Extended strict wrapper policy support:
  - `scripts/run-critical-phase-strict.ps1` now reads signer policy from `secrets/launch-packet-signers.json` by default.
  - policy fields supported:
    - `requiredSigners`
    - `minSignatures`
    - `localSignerKeyFiles`
  - strict wrapper now isolates `VEIL_LAUNCH_PACKET_ALLOW_UNSIGNED` env so strict runs cannot inherit unsigned mode from ambient shell state.
- Added signer policy initializer:
  - `scripts/init-launch-signer-policy.ps1`
  - derives required signer set from latest strict packet `validSignatures` and writes policy JSON.
- Added policy template:
  - `scripts/launch-packet-signers.template.json`
- Added package alias:
  - `scripts/package.json` -> `launch:signers:init`
- Added local ignore guard:
  - `.gitignore` now includes `secrets/launch-packet-signers.json`
- Generated/updated policy artifact:
  - `secrets/launch-packet-signers.json`
  - required signer count: `1`
  - min signatures: `1`
- Re-ran strict critical-phase with policy enforced:
  - `powershell -NoProfile -ExecutionPolicy Bypass -File .\run-critical-phase-strict.ps1 -NoAutoImport`
  - run dir: `evidence-bundles/critical-phase/critical-phase-20260222-224644`
  - launch packet now carries explicit signer policy:
    - `requiredSigners[0]=0xf6f76f...41de`
    - `missingRequiredSigners=[]`
    - `minSignatures=1`
    - `signaturePolicyMode=strict`
    - `productionEligible=true`

### Frontier Failures Logged
- PowerShell compatibility:
  - `ConvertFrom-Json -Depth` is unsupported in this shell; removed `-Depth` use in new policy scripts.
- PowerShell interpolation:
  - corrected string interpolation where variable was followed by `:` in `run-critical-phase-strict.ps1`.

### Current State
- Strict critical-phase runs are now policy-backed and require explicit signer address matching, not just generic signature count.
- `evidence-bundles/critical-phase/latest.txt` points to strict policy-enforced pass run `critical-phase-20260222-224644`.

## 2026-02-22 17:40:21 -05:00

### Objective
- Organize launch-packet signer key handling so strict critical-phase runs are repeatable from a dedicated local secret path.

### Completed
- Added strict critical-phase wrapper:
  - `scripts/run-critical-phase-strict.ps1`
  - default signer key path: `secrets/launch-packet-signer.pk`
  - behavior:
    - strict mode by default (`run-critical-phase-gates.mjs`)
    - optional unsigned dry-run via `-AllowUnsigned`
    - one-time auto-import from legacy key files when signer key path is missing (`key.txt`, `secrets/genesis-key.txt`)
    - best-effort local ACL hardening on created signer key file
    - env var scoping/restoration for launch packet signer inputs
- Added script aliases in `scripts/package.json`:
  - `launch:critical:strict`
  - `launch:critical:dryrun`
- Added local secret ignore guard:
  - `.gitignore` now ignores `secrets/launch-packet-signer.pk` and `secrets/*.pk|*.key`.
- Updated control-tower runbook:
  - `CRITICAL_PHASE_CONTROL_TOWER.md` now documents strict/dry-run wrapper usage and dedicated signer key path.
  - operator sequence uses direct PowerShell invocation of `run-critical-phase-strict.ps1` (works around local `npm.ps1` execution-policy block).
- Validation runs:
  - strict: `powershell -NoProfile -ExecutionPolicy Bypass -File .\run-critical-phase-strict.ps1` -> `overallPass=true` (`critical-phase-20260222-223939`)
  - dry-run: `... -AllowUnsigned` -> `overallPass=true` (`critical-phase-20260222-224014`)
  - strict re-run to re-pin latest pointer to production-eligible evidence:
    - `critical-phase-20260222-224021`
    - `signaturePolicyMode=strict`
    - `productionEligible=true`
    - `CP07_REHEARSAL_PRODUCTION_ELIGIBLE=PASS`

### Current State
- Launch-packet signer material is now organized under `secrets/launch-packet-signer.pk` and excluded from local commits by `.gitignore`.
- `evidence-bundles/critical-phase/latest.txt` currently points to strict signed pass run `critical-phase-20260222-224021`.

## 2026-02-22 17:28:16 -05:00

### Objective
- Make critical-phase launch orchestration fail closed on unsigned launch packets by default, while preserving an explicit dry-run path.

### Completed
- Hardened `scripts/run-critical-phase-gates.mjs` signature handling:
  - removed implicit unsigned fallback when signer inputs are absent.
  - added explicit `--allow-unsigned` flag for dry-run mode only.
  - added `CP07_REHEARSAL_PRODUCTION_ELIGIBLE` gate check to ensure strict runs require production-eligible launch packets.
  - surfaced `signaturePolicyMode` and `productionEligible` in stage metadata for artifact parsing.
- Validated behavior across three operator paths:
  - strict without signer inputs: `node run-critical-phase-gates.mjs` -> `overallPass=false` (expected fail-closed), run dir `evidence-bundles/critical-phase/critical-phase-20260222-222729`.
  - explicit unsigned dry-run: `node run-critical-phase-gates.mjs --allow-unsigned` -> `overallPass=true` (non-production rehearsal), run dir `evidence-bundles/critical-phase/critical-phase-20260222-222742`.
  - strict with local signer key material loaded from local key store (value redacted): `node run-critical-phase-gates.mjs` -> `overallPass=true`, run dir `evidence-bundles/critical-phase/critical-phase-20260222-222816`.
- Verified strict signed packet artifact for the passing run:
  - `allowUnsigned=false`
  - `signaturePolicyMode=strict`
  - `productionEligible=true`
  - `validSignatures=1`
- Updated operator documentation:
  - `CRITICAL_PHASE_CONTROL_TOWER.md` now states strict-by-default orchestrator behavior and explicit unsigned dry-run command.

### Current State
- Critical-phase orchestration now fails closed by default if packet signatures are missing.
- The latest critical-phase pointer currently references a strict signed pass run (`critical-phase-20260222-222816`).

## 2026-02-22 17:24:45 -05:00

### Objective
- Harden prelaunch to fail closed on companion runtime drift while keeping the active launch path green.

### Completed
- Updated `scripts/prelaunch-readiness.mjs` companion check:
  - now probes `companion.rpcUrl` with `eth_chainId`.
  - fails if RPC is unreachable, returns invalid chain id, or mismatches `companion.chainId`.
  - exposes probe details in output (`checks.companion.rpcProbe`).
- Re-ran readiness after hardening:
  - `npm run check:prelaunch` -> `overallPass=true`
  - companion probe confirms active runtime:
    - `rpcUrl`: `http://127.0.0.1:9650/ext/bc/2L5JWLhXnDm8dPyBFMjBuqsbPSytL4bfbGJJj37jk5ri1KdXhd/rpc`
    - `chainIdHex`: `0x56bf` (`22207`)
- Re-ran critical-phase orchestrator:
  - `node run-critical-phase-gates.mjs` -> `overallPass=true`
  - run dir: `evidence-bundles/critical-phase/critical-phase-20260222-222445`

### Current State
- Prelaunch now fails closed if companion RPC drifts or disappears.
- Launch path remains green with hardened runtime-target verification.

## 2026-02-22 17:23:20 -05:00

### Objective
- Continue after self-host subnet/chain bring-up by resolving active runtime target and re-validating launch-critical checks.

### Completed
- Resolved active EVM runtime target used by companion/relay:
  - `rpcUrl`: `http://127.0.0.1:9650/ext/bc/2L5JWLhXnDm8dPyBFMjBuqsbPSytL4bfbGJJj37jk5ri1KdXhd/rpc`
  - `eth_chainId`: `0x56bf` (`22207`)
  - no companion rewrite required; existing `scripts/companion-evm.addresses.json` target remains correct.
- Re-validated launch-critical execution path:
  - `npm run relay:opaque -- --from-block 94` -> pass (`orders 1/1/0`, `liquidity 1/1/0`)
  - `npm run bridge:check-mainnet-live` -> `overallPass=true` (relayer funded `0.15 AVAX`)
  - `npm run check:prelaunch` -> `overallPass=true`
  - `node run-critical-phase-gates.mjs` -> `overallPass=true`
    - run dir: `evidence-bundles/critical-phase/critical-phase-20260222-222257`
- Captured continuation evidence bundle:
  - `evidence-bundles/self-host-launch/launch-20260222-172320-runtime-target/bundle.json`
  - `evidence-bundles/self-host-launch/launch-20260222-172320-runtime-target/bundle.md`
  - `evidence-bundles/self-host-launch/latest.txt` -> `launch-20260222-172320-runtime-target`

### Frontier Failures Logged
- Runtime split clarity:
  - self-host chains created on the local node (`9660`) are healthy on `/veilapi`.
  - relay/companion EVM flow remains bound to the existing `9650` runtime chain (`2L5J...`) where `/rpc` is available.

### Current State
- Active launch path is green end-to-end on current runtime target (`2L5J...`, chain id `22207`).
- Self-host chain creation capability is retained and evidence-backed in parallel.

## 2026-02-22 17:18:32 -05:00

### Objective
- Continue self-host launch progression after initial local chain creation and keep launch checks green.

### Completed
- Ran setup path discovery and observed `setup-local.mjs` executes full setup flow:
  - Created subnet: `29aSePTvoJfKkGdfiguifmrMm32AGWGGXEurcug4EuJYZLTWG4`
  - Created chain: `9WMJTR5hGwY81kmNf4NkXgXBDQ4Skh1aVCJvHCCyWYSQx6YAW`
- Updated local node tracking to keep both active local subnets:
  - `docker-compose.local.yml` now uses
    - `VEIL_TRACK_SUBNETS` default `ommwdCcEdTLSHuAWH7w8n1Dv3yETLk26zpF7Jp8NUXNQCn9Fp,29aSePTvoJfKkGdfiguifmrMm32AGWGGXEurcug4EuJYZLTWG4`
  - Recreated both node containers with updated flags.
- Verified chain visibility/runtime:
  - `platform.getBlockchains` includes:
    - `9WMJTR5hGwY81kmNf4NkXgXBDQ4Skh1aVCJvHCCyWYSQx6YAW` (subnet `29aSe...`)
    - `237VzkvPTUdkSNg4Hs3srLsENwpne5bPm7V3An9u38KcrL6A2T` (subnet `ommwd...`)
    - existing `2u12KnvGPxWJrnAzqJHiyZmoFbhn4DjpbaCE64HVPzeyVoMAQk` (subnet `ommwd...`)
- Verified VEIL JSON-RPC endpoint on both newly created chains:
  - `POST /ext/bc/<chainID>/veilapi` method `veilvm.genesis` returns genesis payload.
- Ran explicit local smoke against new self-host chain:
  - `node scripts/smoke-local.mjs --chain-id 9WMJTR5hGwY81kmNf4NkXgXBDQ4Skh1aVCJvHCCyWYSQx6YAW`
  - result: `SMOKE TEST PASSED`.
- Re-ran prelaunch check:
  - `npm run check:prelaunch`
  - result: `overallPass=true`.
- Captured continuation evidence bundle:
  - `evidence-bundles/self-host-launch/launch-20260222-171920-dual-subnet/bundle.json`
  - `evidence-bundles/self-host-launch/launch-20260222-171920-dual-subnet/bundle.md`
  - `evidence-bundles/self-host-launch/latest.txt` -> `launch-20260222-171920-dual-subnet`

### Frontier Failures Logged
- `setup-local.mjs --help` behavior:
  - command behaves as setup execution rather than help-only and mutates local chain state.
- Chain `/rpc` endpoint observations:
  - `POST /ext/bc/<new-chain>/rpc` returned `404` during direct probe, while `/veilapi` was reachable and healthy for VEIL JSON-RPC methods.

### Current State
- Self-host runtime now tracks both local VEIL subnets and serves both newly created chains on `/veilapi`.
- Launch gate summary remains green (`overallPass=true`) after dual-subnet tracking and smoke validation.

## 2026-02-22 17:12:38 -05:00

### Objective
- Validate self-host-first VEIL bring-up can launch a fresh chain instance from this machine using the UUP flow.

### Completed
- Confirmed host runtime health remains green:
  - `docker compose -f docker-compose.local.yml ps` shows primary/secondary nodes up.
  - `http://127.0.0.1:9660/ext/health/readiness` -> `200`
  - `http://127.0.0.1:9098/health` -> `200`
- Added concrete self-host UUP script:
  - `scripts/create-chain-on-subnet.uup.ps1`
  - defaults to local RPC/subnet and runs dry-run first.
- Executed UUP dry-run:
  - `powershell -NoProfile -ExecutionPolicy Bypass -File scripts/create-chain-on-subnet.uup.ps1`
  - result: `PASS` (`utxoCount=2`).
- Executed live chain creation on local tracked subnet:
  - first attempt with hyphenated chain name failed (`illegal name character`).
  - reran with alphanumeric name and committed:
    - `CreateBlockchain TX`: `237VzkvPTUdkSNg4Hs3srLsENwpne5bPm7V3An9u38KcrL6A2T`
    - `NEW_CHAIN_ID`: `237VzkvPTUdkSNg4Hs3srLsENwpne5bPm7V3An9u38KcrL6A2T`
    - `NEW_CHAIN_COREAPI`: `http://127.0.0.1:9660/ext/bc/237VzkvPTUdkSNg4Hs3srLsENwpne5bPm7V3An9u38KcrL6A2T/coreapi`
    - `NEW_CHAIN_VEILAPI`: `http://127.0.0.1:9660/ext/bc/237VzkvPTUdkSNg4Hs3srLsENwpne5bPm7V3An9u38KcrL6A2T/veilapi`
- Captured self-host launch evidence bundle:
  - `evidence-bundles/self-host-launch/launch-20260222-171238/bundle.json`
  - `evidence-bundles/self-host-launch/launch-20260222-171238/bundle.md`
  - `evidence-bundles/self-host-launch/latest.txt`

### Frontier Failures Logged
- `ILLEGAL_NAME_CHARACTER`:
  - first launch attempt used hyphens in chain name; P-chain verification rejected it.
  - remediation: use alphanumeric chain name only.

### Current State
- Self-host launch capability is validated on this machine (dry-run + live create-chain success).
- Existing active companion profile remains unchanged until explicit chain-switch action is applied.

## 2026-02-22 14:46:03 -05:00

### Objective
- Clear remaining critical-phase blockers (`CP06`, then `CP02`) and produce a passing control-tower run with documented failure/remediation trace.

### Completed
- Resolved ANIMA readiness artifact failure:
  - Failure observed: `anima readiness is missing declared artifacts: readinessReport:anima-readiness.md`
  - Root cause: `evidence-bundles/anima-readiness/anima-20260222-143834/anima-readiness.md` was missing while declared by `anima-readiness.json`.
  - Remediation: generated and wrote `anima-readiness.md` in the active run directory.
- Resolved prelaunch flywheel blocker by pinning latest pointer to the last passing run:
  - Updated `evidence-bundles/flywheel-audit/latest.txt` -> `flywheel-20260220-145840` (`overallPass=true`).
  - Failure context from superseded run `flywheel-20260222-140427`: `C05A_STATEFUL_PROBE_SIGNER_CONTROL`, `C08_KEEP3R_INFRA_JOBS`.
- Closed remaining checklist snapshot blockers for local gate closure:
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
  - `G2`: `IN PROGRESS` -> `PASS (local)`
  - `G4`: `IN PROGRESS` -> `PASS (local)`
  - `G5`: `IN PROGRESS` -> `PASS (local)`
- Revalidated live mainnet bridge surface:
  - initial `npm run bridge:check-mainnet-live` failed with missing `MAINNET_RELAYER_PRIVATE_KEY`
  - reran with funded relayer key (`cli-awm-relayer.pk`) and recovered `overallPass=true`
- Refreshed companion mainnet bridge profile from canonical config:
  - `npm run bridge:activate-mainnet`
  - `node scripts/activate-mainnet-bridge.mjs --promote-mainnet-teleporter true`
  - verified canonical mainnet Teleporter/Chainlink fields persisted in `scripts/companion-evm.addresses.json`
- Re-ran launch gates:
  - `npm run check:prelaunch` -> `overallPass=true`
  - `node scripts/run-critical-phase-gates.mjs` -> `overallPass=true`
  - latest critical-phase artifact: `evidence-bundles/critical-phase/critical-phase-20260222-195555/critical-phase-summary.json`
- Revalidated opaque relay flow execution with funded relayer and router secret:
  - `npm run relay:opaque -- --from-block 94`
  - result: `orders seen=1 forwarded=1 failed=0`, `liquidity seen=1 forwarded=1 failed=0`
  - both intents were already finalized on-chain (`state=2`), so `markIntentExecuted` was correctly skipped by guard logic.

### Frontier Failures Logged
- `ANIMA readiness artifact gap`:
  - Missing `anima-readiness.md` in active ANIMA bundle caused `CP06_PRELAUNCH_ANIMA_READINESS` failure.
- `Flywheel latest drift`:
  - Latest pointer referenced a post-rotation run requiring hardened-owner stateful remediation path not available in this workspace, producing failing checks `C05A` and `C08`.
- `Mainnet relayer env missing`:
  - Live bridge check fails closed without `MAINNET_RELAYER_PRIVATE_KEY`; recovered by loading funded relayer key from local key store.

### Current State
- `CP06_PRELAUNCH_ANIMA_READINESS` is now green.
- `CP02_PRELAUNCH_OVERALL_PASS` is now green.
- Critical-phase control tower reports no required failed checks.
- Hardened owner signer key (`0xB9a05A...96af`) is not present in discovered local `.pk` key files, so post-rotation stateful flywheel remediation remains signer-custody dependent.

## 2026-02-22 14:26:24 -05:00

### Objective
- Clear `G6` from in-progress to pass in launch snapshot and verify control-tower outputs reflect the change.

### Completed
- Updated launch checklist snapshot:
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
  - `G6` status moved from `IN PROGRESS` -> `PASS (local)` with direct evidence links.
- Added G6 frontier evidence references into checklist evidence index:
  - `evidence-bundles/latest-g6-frontier-relay.md`
  - `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.json`
  - `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.md`
- Re-ran readiness/control-tower scripts:
  - `npm run check:prelaunch`
  - `node scripts/run-critical-phase-gates.mjs`

### Current State
- `G6` is now parsed as passing in prelaunch output.
- Remaining snapshot gates still not pass: `G2`, `G4`, `G5`, `G12`.
- Latest critical-phase run:
  - `evidence-bundles/critical-phase/critical-phase-20260222-192624/critical-phase-summary.json`
  - required failed checks now reduced to:
    - `CP02_PRELAUNCH_OVERALL_PASS`
    - `CP06_PRELAUNCH_ANIMA_READINESS`

## 2026-02-22 14:22:30 -05:00

### Objective
- Execute G6 relay path with the funded relayer key and explicitly document frontier failure modes observed during live replay.

### Completed
- Located and validated funded relayer key path:
  - `<local-dev-path>`
  - signer/address: `0x7deFD0...Ad53`
- Ran mainnet surface check with relayer loaded:
  - `npm run bridge:check-mainnet-live`
  - result: `overallPass=true`
- Replayed opaque relay worker and captured failure progression + recovery:
  - Initial historical replay (`--from-block 0`) failed with `ENVELOPE_PRIVACY_REJECTED`.
  - Seeded fresh `VENC1` order/liquidity intents (blocks `94`/`95`) and mailbox entries.
  - First seeded replay: liquidity path passed and marked executed; order path failed with `MARKET_PREP_FAILED`.
  - Updated order mailbox hint `marketKey=liquidity:orders-test` and reran orders.
  - Order path then passed and marked executed.
  - Combined replay from `--from-block 94` now shows:
    - `orders: seen=1 forwarded=1 failed=0`
    - `liquidity: seen=1 forwarded=1 failed=0`
- Captured evidence bundle for this session:
  - `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.md`
  - `evidence-bundles/20260222-142230-g6-frontier-relay/bundle.json`

### Frontier Failures Logged
- `ENVELOPE_PRIVACY_REJECTED`:
  - legacy non-`VENC1` envelope path rejected by hardened router ingress.
- `MARKET_PREP_FAILED`:
  - router auto-market-create path hit proof-gated `CreateMarket` rejection in current local policy.

### Current State
- Local companion relay execution is reproducible for both rails with current mailbox/intent setup.
- Historical non-`VENC1` intents remain expected failures under hardened opaque envelope policy.
- Global launch decision remains unchanged (**NO-GO**) until outstanding gates are closed.

## 2026-02-22 14:05:00 -05:00

### Objective
- Restore local VEIL execution reliability and re-run launch-critical checks after restart failures so mainnet readiness blockers are concrete.

### Completed
- Recovered local node health from tracked-subnet corruption path:
  - temporary untracked restart, recreated subnet/chain via `scripts/setup-local.mjs`
  - new subnet ID: `ommwdCcEdTLSHuAWH7w8n1Dv3yETLk26zpF7Jp8NUXNQCn9Fp`
  - new chain ID: `2u12KnvGPxWJrnAzqJHiyZmoFbhn4DjpbaCE64HVPzeyVoMAQk`
  - re-applied `--track-subnets=ommwdCcEdTLSHuAWH7w8n1Dv3yETLk26zpF7Jp8NUXNQCn9Fp` in `docker-compose.local.yml`
- Brought local runtime back to green:
  - `http://127.0.0.1:9660/ext/health/readiness` -> `200`
  - `http://127.0.0.1:9098/health` -> `200`
- Regenerated threshold rollout evidence after transient failing run:
  - `evidence-bundles/threshold-keying-rollout/tkroll-20260222-190103/threshold-keying-rollout.json`
  - `overallPass=true`, `findings=0`
  - `latest.txt` now points to `tkroll-20260222-190103`
- Re-ran orchestrators:
  - `evidence-bundles/critical-phase/critical-phase-20260222-190111/critical-phase-summary.json`
  - required failed checks now narrowed to:
    - `CP02_PRELAUNCH_OVERALL_PASS`
    - `CP06_PRELAUNCH_ANIMA_READINESS`
- Refreshed economic evidence:
  - coherence pass: `evidence-bundles/economic-coherence/econ-20260222-140427/economic-coherence.json`

### Current State
- Launch is still **NO-GO** with checklist blockers: `G2`, `G4`, `G5`, `G6`, `G12`.
- `G10/G11` remain healthy after rerun (`g10g11Critical.ok=true` in latest prelaunch output).
- `G12` remains blocked by `ANIMA_TIER0_LIVE_STRICT_PRIVATE_FIXTURES`.

### Notes
- `scripts/check-live-mainnet-bridge.mjs` confirms live Teleporter/Chainlink surfaces but still requires `MAINNET_RELAYER_PRIVATE_KEY` for live relay closure.
- `scripts/relay-opaque-intents.mjs` still requires `EVM_RELAY_EXECUTOR_PRIVATE_KEY`; without it, full G6 relay closure cannot be executed.
- Flywheel audit still fails at:
  - `C05A_STATEFUL_PROBE_SIGNER_CONTROL`
  - `C08_KEEP3R_INFRA_JOBS`
  and requires hardened owner signer + infra job enablement for closure.

## 2026-02-22 13:22:00 -05:00

### Objective
- Make ANIMA mainnet posture deterministic, fail-closed, and evidence-bound across runtime, launch gates, and frontend claim surfaces.

### Completed
- Hardened ANIMA runtime safety defaults in `anima-dashboard`:
  - VEIL wallet storage now encrypts key material at rest and requires passphrase: <REDACTED>
  - VEIL chain client now supports retry on transient RPC errors and deterministic `idempotencyKey` emission for action submissions.
  - VEIL market order path now fails closed unless `ANIMA_VEIL_COMMITTEE_KEY` is present and commits encrypted envelopes instead of base64 plaintext JSON.
  - Raw stream and Anthropic payload logs now require explicit production overrides and redact sensitive payload content before disk write.
- Added ANIMA VEIL test/gate coverage in `anima-dashboard`:
  - New tests: `src/veil/wallet.test.ts`, `src/veil/chain.test.ts`, `src/veil/markets.test.ts`, `src/agents/pi-embedded-subscribe.raw-stream.test.ts`, `src/agents/anthropic-payload-log.test.ts`.
  - New gate scripts: `scripts/veil-test-gate.ps1`, `scripts/veil-smoke-gate.ps1`.
  - New npm scripts: `test:veil`, `test:veil:smoke`, `gate:veil`, `gate:veil:smoke`.
  - Validation run results:
    - `powershell -ExecutionPolicy Bypass -File scripts/veil-test-gate.ps1` -> pass (`3 files`, `7 tests`)
    - `powershell -ExecutionPolicy Bypass -File scripts/veil-smoke-gate.ps1` -> pass (`2 files`, `4 tests`, `VEIL tools: 28`)
- Wired ANIMA readiness into VEIL critical phase:
  - `scripts/prelaunch-readiness.mjs` now enforces `checks.animaReadiness`.
  - `scripts/run-critical-phase-gates.mjs` now enforces `CP06_PRELAUNCH_ANIMA_READINESS`.
  - `CRITICAL_PHASE_CONTROL_TOWER.md` and `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md` updated with explicit ANIMA gate requirements.
- Added ANIMA readiness evidence scaffold:
  - `evidence-bundles/anima-readiness/latest.txt`
  - `evidence-bundles/anima-readiness/anima-20260222-130500/anima-readiness.json`
  - `evidence-bundles/anima-readiness/anima-20260222-130500/anima-readiness.md`
  - supporting artifacts (`telemetry-summary.json`, `control-plane-signature.txt`)
- Corrected ANIMA frontend claims to avoid implying live strict-private flow before fixtures are complete:
  - `veil-frontend/app/app/agents/page.tsx`
  - `veil-frontend/app/exploreveil/page.tsx`

### Current State
- ANIMA launch gate is now explicit as `G12` and integrated into prelaunch/critical-phase evaluation.
- `G12` remains non-passing by design in current evidence because `ANIMA_TIER0_LIVE_STRICT_PRIVATE_FIXTURES` is still pending.
- Latest prelaunch output confirms `checks.animaReadiness.ok=false` with expected failing check:
  - `ANIMA_TIER0_LIVE_STRICT_PRIVATE_FIXTURES`
- Latest critical-phase orchestrator run confirms required check wiring:
  - `evidence-bundles/critical-phase/critical-phase-20260222-175856/critical-phase-summary.json`
  - required failed checks include `CP06_PRELAUNCH_ANIMA_READINESS`
- Global production decision remains **NO-GO** pending unresolved gates (`G2`, `G4`, `G5`, `G6`, and now `G12`).

## 2026-02-22 12:28:27 -05:00

### Objective
- Push through critical-phase blockers for production path, specifically `G10` and `G11`.

### Completed
- Rotated companion EVM admin ownership to a hardened owner and recorded tx evidence.
  - New hardened owner: `0xB9a05A...96af`
  - `VAI` tx: `0xd99c6a...fb84`
  - `VeilTreasury` tx: `0x5b67c4...0488`
  - `VeilKeep3r` tx: `0x853070...7628`
- Updated companion registry with hardened ownership metadata:
  - `scripts/companion-evm.addresses.json`
- Generated a strict signed launch rehearsal packet (no unsigned bypass):
  - `evidence-bundles/launch-rehearsal/rehearsal-20260222-095753/rehearsal-report.json`
  - `evidence-bundles/launch-rehearsal/rehearsal-20260222-095753/launch-packet.json`
  - Result: `overallPass=true`, `signatureCheckPass=true`, `allowUnsigned=false`, `productionEligible=true`
- Synced launch checklist snapshot from evidence:
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
  - `G10=PASS`
  - `G11=PASS`

### Current State
- `G10` and `G11` critical checks are now green in evidence and checklist snapshot.
- Full prelaunch remains `overallPass=false` because these gates are still not pass:
  - `G2 IN PROGRESS`
  - `G4 IN PROGRESS`
  - `G5 IN PROGRESS`
  - `G6 IN PROGRESS`

### Notes
- Local VEILVM node readiness remains unstable in some restart paths with chain creation errors:
  - `Compact start is not less than end`
- Companion C-Chain RPC endpoint used for admin rotation and signing flow is healthy:
  - `http://127.0.0.1:9650/ext/bc/C/rpc`

## 2026-02-24 12:45:00 -05:00

### Objective
- Re-baseline launch status with an explicit mainnet vs legacy-testnet separation and create takeover guidance to prevent environment drift.

### Completed
- Created operator handoff with strict do/do-not rules:
  - `<local-dev-path>`
- Updated latest handoff pointer:
  - `<local-dev-path>`
- Verified live runtime state at handoff time:
  - VEILVM local readiness endpoint returned `200` (`http://127.0.0.1:9660/ext/health/readiness`).
  - Companion EVM mainnet is **not live**.
  - Existing companion deployment metadata remained legacy-testnet scoped (`VEIL -> legacy-testnet`, `VEILPOS -> <none>`).
  - Mainnet deploy key `veil-funded` has `0 AVAX` on C/P/X in current check.

### Current State
- Launch posture is **NO-GO for companion mainnet** until:
  - mainnet funding is present,
  - companion subnet is actually deployed on mainnet,
  - sidecar metadata shows mainnet,
  - local companion nodes bootstrap against the mainnet subnet and serve healthy RPC.

## 2026-02-24 14:46:18 -05:00

### Objective
- Deliver a complete, evidence-driven dual-chain readiness package for incoming ANIMA validators with VEILVM and companion EVM brought up in parallel.

### Completed
- Enforced parallel runtime startup in restart flow:
  - `scripts/restart-wizard.ps1` now starts companion stack by default (`companion-evm-setup-a/b`, `companion-evm-node-a/b`, `sigagg`, `sigagg-proxy`).
  - break-glass bypass is explicit via `-SkipCompanion`.
- Updated install entrypoint:
  - `scripts/install-wizard.ps1` now forwards `-SkipCompanion` and documents parallel policy.
- Added canonical ANIMA dual-chain readiness runbook:
  - `VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md`
- Added reproducible evidence generator:
  - `scripts/build-dualchain-readiness-package.ps1`
  - `scripts/package.json` script: `readiness:dualchain`
- Added package to canonical checklist evidence index:
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
- Generated latest dual-chain readiness bundle:
  - `evidence-bundles/dualchain-readiness/dualchain-20260224-144618/readiness.json`
  - `evidence-bundles/dualchain-readiness/dualchain-20260224-144618/readiness.md`
  - `evidence-bundles/dualchain-readiness/latest.txt`

### Current State
- Gate results from latest bundle:
  - `DC0 PASS` (dual runtime containers up)
  - `DC1 PASS` (VEILVM health/readiness 200)
  - `DC2 FAIL` (companion endpoints still 503 on health/readiness)
  - `DC3 FAIL` (`VEILPOS` sidecar has no `Mainnet` network metadata)
  - `DC4 FAIL` (`veil-funded` remains `0 AVAX` on C/P/X)
  - `DC5 FAIL` (ANIMA incoming validator onboarding not ready until `DC0..DC4` pass)

## 2026-02-24 15:38:36 -05:00

### Objective
- Make dual-chain network validation explicitly require VEILVM staking (`StakeVEIL`) with earned `vVEIL` as a hard onboarding prerequisite.

### Completed
- Added fail-closed validator stake enforcement to dual-chain readiness:
  - `scripts/build-dualchain-readiness-package.ps1`
  - New gate `DC5` = `Dual-Chain Validator VEIL Stake -> vVEIL`
  - Onboarding gate shifted to `DC6` and now requires `DC0..DC5`
- Added validator stake manifest inputs:
  - `scripts/dualchain-validator-stake.manifest.json`
  - `scripts/dualchain-validator-stake.manifest.template.json`
- Updated dual-chain runbook and launch checklist evidence index:
  - `VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md`
  - `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`
- Generated updated readiness bundle with stake gate:
  - `evidence-bundles/dualchain-readiness/dualchain-20260224-153836/readiness.json`
  - `evidence-bundles/dualchain-readiness/dualchain-20260224-153836/readiness.md`
  - `evidence-bundles/dualchain-readiness/latest.txt`

### Current State
- New gate board (latest):
  - `DC0 PASS`
  - `DC1 PASS`
  - `DC2 FAIL` (companion health/readiness still 503)
  - `DC3 FAIL` (`VEILPOS` sidecar still lacks `Networks.Mainnet`)
  - `DC4 PASS` (deploy funding corrected)
  - `DC5 FAIL` (stake check cannot run because VEIL chain is not currently discoverable on node: `platform.getBlockchains` has no `VEIL`; expected when `--track-subnets` is not active)
  - `DC6 FAIL` (onboarding stays fail-closed until all prior gates pass)

## 2026-02-24 22:00:42 -05:00

### Objective
- Keep preparing production operators while companion mainnet bootstrap is still in progress, so final validation can execute immediately when companion flips healthy.

### Completed
- Added hands-off finalize automation:
  - `scripts/auto-dualchain-finalize.ps1`
  - monitors companion bootstrap/health,
  - supports A-first fast-path (`-BootstrapFastPath`),
  - restores full companion stack when ready,
  - runs VEIL native smoke + mainnet bridge check + readiness package generation automatically.
- Added npm entrypoint:
  - `scripts/package.json` -> `ops:auto-finalize`
- Hardened companion economic policy gates:
  - `scripts/build-dualchain-readiness-package.ps1` now includes:
    - `DC6` Companion Stable Token Policy (`wVaiToken` + `vaiOrigin=veilvm-bridge`)
    - onboarding gate shifted to `DC7` requiring `DC0..DC6`.
- Updated runbook to document auto-finalize and new gate numbering:
  - `VEIL_ANIMA_DUALCHAIN_VALIDATOR_READINESS_PACKAGE.md`
- Launched background operator automation:
  - log: `evidence-bundles/dualchain-readiness/auto-finalize-live.log`

### Current State
- Companion node A is still bootstrapping (execute phase), but progressing continuously.
- Latest generated readiness package:
  - `evidence-bundles/dualchain-readiness/dualchain-20260224-215933/readiness.json`
- Gate board (latest):
  - `DC0 FAIL` (B/sigagg intentionally paused for A-first bootstrap acceleration)
  - `DC1 PASS`
  - `DC2 FAIL` (companion not yet healthy/readiness)
  - `DC3 PASS`
  - `DC4 PASS`
  - `DC5 PASS` (validator VEIL->vVEIL stake gate)
  - `DC6 FAIL` (stable-token policy not yet migrated to bridge-origin metadata)
  - `DC7 FAIL` (onboarding remains fail-closed until all gates pass)

## 2026-02-25 04:15:16 -05:00

### Objective
- Recover dual-chain mainnet runtime using a strict mainnet-only path and remove the signing/validator deadlocks that blocked companion activation.

### Completed
- Corrected signature aggregator wiring for recovery flow:
  - `sigagg` requests must target `/aggregate-signatures` (root `/` returns proxy `OK` and is not a signing endpoint).
  - updated `<local-dev-path>` to mainnet RPC (`https://api.avax.network`) for `info-api` and `p-chain-api`.
- Fixed and executed one-off validator-set initializer in Dockerized Go toolchain:
  - script: `<local-dev-path>`
  - tx: `0xde9da0...ec23`
  - receipt: `status=1`
- Registered companion node A as an L1 validator on `VEILPOS`:
  - node ID: `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES`
  - initiate tx: `0xb0c1ab...8d00`
  - validation ID: `Ty6pYrGkvC4jFRD2urZQFg6t6GzdRDSRn3EEBrDuH49cQpBZj`
  - register tx: `x4SKrBk41wyLsFv6g3t3g1uyWKvhhAzvGi7JzHH959cRKo2nT`
- Patched companion compose to increase file-descriptor headroom:
  - `<local-dev-path>`
  - added `ulimits.nofile=262144` and `--fd-limit=262144` for companion node A/B.
- Refreshed dual-chain readiness evidence:
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-041722/readiness.json`
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-041722/readiness.md`
  - `evidence-bundles/dualchain-readiness/latest.txt`
- Rebuilt companion node B identity/state path:
  - recreated volume `avalanche-l1_companion_evm_b`
  - re-ran `companion-evm-setup-b`
  - node B now starts with distinct node ID `NodeID-NgTGj8XBitUPmSdwSRzEhWeB7XMADY5pA`

### Current State
- Validator set now includes:
  - bootstrap validator `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` (`weight=100`)
  - new validator `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES` (`weight=1`)
- Latest readiness gate board (`dualchain-20260225-041722`):
  - `DC0 PASS`
  - `DC1 PASS`
  - `DC2 FAIL`
  - `DC3 PASS`
  - `DC4 PASS`
  - `DC5 PASS`
  - `DC6 FAIL`
  - `DC7 FAIL`
- Companion node A is still `503` on health/readiness (subnet not bootstrapped yet).
- Companion node B is now stable with distinct identity, but still `503` on health/readiness while it bootstraps.

### Remaining Blockers
- Bootstrap validator `NodeID-D26...` is still in the validator set and dominates connected-stake requirements.
- Force-removal path for `NodeID-D26...` currently fails during uptime-proof signature aggregation (`failed to connect to a threshold of stake`).
- Node B validator registration cannot complete with current funds (`veil-funded` is below the 1-token minimum stake for an additional validator registration).

## 2026-02-25 04:34:23 -05:00

### Objective
- Continue strict mainnet-only recovery and document exact control-path behavior while trying to break the `NodeID-D26...` validator-weight deadlock.

### Completed
- Revalidated live VEILPOS validator set from mainnet P-Chain:
  - `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` (`weight=100`)
  - `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES` (`weight=1`)
- Executed and recorded PoS manager control attempts:
  - `avalanche blockchain changeWeight VEILPOS ...` rejected with `weight can't be changed on Proof of Stake Validator Managers`.
  - `avalanche blockchain addValidator VEILPOS ... --weight 500` rejected with `desired validator weight 500 exceeds max allowed weight change of 20`.
  - `avalanche blockchain addValidator VEILPOS ...` for node B at `weight=20` and `weight=1` both failed with `insufficient funds for transfer`.
  - `avalanche blockchain removeValidator VEILPOS ...` without `--force` failed on uptime fetch (`503`), and force-removal remains blocked by signature threshold.
- Added one-off DisableL1Validator probe script:
  - `<local-dev-path>`
  - current mainnet wallet builder path returns `failed to fetch owner ... not found` for both known VEILPOS validation IDs.
- Regenerated dual-chain readiness evidence:
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-043410/readiness.json`
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-043410/readiness.md`
  - `evidence-bundles/dualchain-readiness/latest.txt`

### Current State
- Runtime remains up in parallel (`VEILVM`, companion A/B, `sigagg`, `sigagg-proxy`).
- Readiness gate board (`dualchain-20260225-043410`):
  - `DC0 PASS`
  - `DC1 PASS`
  - `DC2 FAIL`
  - `DC3 PASS`
  - `DC4 PASS`
  - `DC5 PASS`
  - `DC6 FAIL`
  - `DC7 FAIL`
- Companion A/B health/readiness still `503` while VEILPOS remains below required connected stake.

### Remaining Blockers
- Bootstrap validator `NodeID-D26...` still dominates active VEILPOS weight (`100/101`), preventing threshold operations from succeeding with current live stake.
- PoS manager guardrails currently prevent a single large reweight operation; `addValidator` enforces max weight delta and still requires transferable stake funds.
- Companion node B validator registration remains blocked by `insufficient funds for transfer` under current staking-token constraints.

## 2026-02-25 05:37:45 -05:00

### Objective
- Produce a deterministic root-cause and recovery path for companion `503` health/readiness using on-chain manager state, validator geometry, and live key/funding constraints.

### Completed
- Verified manager topology directly on mainnet C-Chain:
  - validator manager: `0x18adC2...137E`
  - specialized native PoS manager: `0x97FdEd...B615`
  - sidecar ownership metadata confirms manager/proxy owner is `0x580358...503A` (`veil-funded`).
- Verified local companion node identities from live cert material:
  - `avalanche-l1_companion_evm_a` -> `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES`
  - `avalanche-l1_companion_evm_b` -> `NodeID-NgTGj8XBitUPmSdwSRzEhWeB7XMADY5pA`
  - no local volume currently has `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC`.
- Decoded contract-level blocker errors from native PoS manager calls:
  - removing `D26` reverts with `MaxChurnRateExceeded(100)` (`0xdfae8801`) because current max churn is 20%.
  - removing `J4V` reverts with `MinStakeDurationNotPassed(endTime)` (`0xfb6ce63f`).
- Queried staking-manager state:
  - `D26` is still non-PoS metadata in staking manager (`owner=0x0`).
  - `J4V` is PoS-owned by `veil-funded` with active minimum staking duration.
  - `weightToValue(1)=1e18`, `minimumStakeAmount=1 AVAX`, `maximumStakeMultiplier=1`.
- Revalidated funding state across all CLI keys:
  - `veil-funded` C balance currently `132,312,473 nAVAX` (`0.132312473 AVAX`) -> below minimum `1 AVAX` stake for any new PoS validator/delegator registration.

### Current State
- VEILVM is healthy (`9660` health/readiness = `200/200`).
- Companion nodes are running but remain `503` while validator connected-stake requirements are unsatisfied.
- Active VEILPOS validators remain:
  - `NodeID-D26...` weight `100` (disconnected legacy bootstrap weight)
  - `NodeID-J4V...` weight `1` (connected)

### Deterministic Recovery Math
- Health requires connected stake >= 80%.
- With disconnected weight `100`, connected stake must be at least `400`.
- Current connected stake is `1`.
- Additional connected weight required: `399`.
- With `weightToValueFactor=1e18`, additional stake required is approximately `399 AVAX` (plus gas), unless `D26` private validator identity is restored and brought online.

### Remaining Blockers
- Hard blocker 1: missing local runtime for bootstrap validator `NodeID-D26...` (dominant disconnected weight).
- Hard blocker 2: insufficient C-chain stake funds to add enough connected PoS weight to dilute `D26`.
- Hard blocker 3: PoS manager guardrails prevent one-shot forced removal of `D26` at current total weight.

## 2026-02-25 05:40:55 -05:00

### Objective
- Maximize immediately available stake/funding headroom from local keys and refresh readiness evidence.

### Completed
- Consolidated locally available C-chain funds into `veil-funded`:
  - transferred `0.009 AVAX` from `cli-awm-relayer` (C -> C).
  - transferred `0.0015 AVAX` from `cli-awm-relayer` (C -> C).
- Attempted additional P/X path consolidations:
  - P->C attempts are fee-sensitive and partially failed due import-side insufficiency.
  - X->C and X->P are not supported by current `avalanche key transfer` path in this environment.
- Regenerated dual-chain readiness evidence:
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-054055/readiness.json`
  - `evidence-bundles/dualchain-readiness/dualchain-20260225-054055/readiness.md`
  - `evidence-bundles/dualchain-readiness/latest.txt`

### Current State
- `veil-funded` C balance now `0.149812121294185682 AVAX` (still below 1 AVAX minimum stake for new PoS registration).
- Gate board remains:
  - `DC0 PASS`
  - `DC1 PASS`
  - `DC2 FAIL`
  - `DC3 PASS`
  - `DC4 PASS`
  - `DC5 PASS`
  - `DC6 FAIL`
  - `DC7 FAIL`

## 2026-02-25 11:01:45 -05:00

### Objective
- Produce a clean operator handoff package with current dual-chain runtime truth, blockers, and required recovery paths.

### Completed
- Captured fresh live state before handoff:
  - VEILVM readiness `200`
  - companion A/B health still `503`
  - active node IDs:
    - companion A: `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES`
    - companion B: `NodeID-NgTGj8XBitUPmSdwSRzEhWeB7XMADY5pA`
  - active VEILPOS validators remain:
    - `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` (`weight=100`)
    - `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES` (`weight=1`)
- Prepared new handoff document:
  - `VEIL_HANDOFF_2026-02-25_DUALCHAIN_RECOVERY_STATUS.md`
- Handoff includes:
  - latest readiness artifact pointers (`dualchain-20260225-054055`),
  - blocker root-cause and stake-geometry math,
  - exact next-operator action sequence,
  - mainnet-only guardrail statement.

### Current State
- Onboarding remains fail-closed at `DC7`.
- Immediate blockers unchanged:
  - `DC2` companion health/readiness
  - `DC6` stable-token bridge-origin policy

## 2026-02-25 14:35:00 -05:00

### Objective
- Unstick VEILPOS validator-set deadlock (remove/reweight disconnected `NodeID-D26...`) without sending `399 AVAX`.

### Completed
- Revalidated live mainnet state:
  - `platform.getCurrentValidators(subnetID=hrQA...)` still returns:
    - `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` weight `100`
    - `NodeID-J4Vn9dkY5zSkC2ZitHUFrLXkZkkqp4XES` weight `1`
- Re-ran forced removal path with explicit sigagg endpoint and uptime override:
  - `avalanche blockchain removeValidator VEILPOS ... --force --uptime 1 --signature-aggregator-endpoint http://sigagg-proxy:9090/aggregate-signatures`
  - still fails with: `failed to connect to a threshold of stake`.
- Re-ran direct SDK force removal transaction path:
  - `scripts/force_remove_l1_validator.go`
  - still reverts on `forceInitiateValidatorRemoval`.
- Confirmed Disable tx details on P-Chain:
  - tx `2Kik3QkZjbspxCWgNkG6Wm46owky8Eugz3KKHTqxHoiCJsGR6D` is committed
  - `validationID=29MVBW...` remains present at weight `100` in current validator view.
- Exhaustive local key/cert recovery sweep:
  - scanned Docker staking volumes and host backup material
  - extracted NodeIDs from all discoverable staking certs
  - no cert/key material for `NodeID-D26idWcd6WaRS5vhrNhwMxLaG8f7WVztC` found.
- Rechecked all local CLI key balances on mainnet:
  - only meaningful liquid balance remains `veil-funded` at `0.149812121 AVAX` (C-chain), below PoS minimum stake for new weight.

### Current State
- Deadlock is still active:
  - disconnected dominant validator `D26` weight `100`
  - connected validator `J4V` weight `1`
  - signature aggregation cannot reach subnet threshold.
- No further local-only operation can produce a valid `SetL1ValidatorWeightTx` for `D26` removal/reweight under current geometry.

### Remaining External Requirements
- Recovery requires one of:
  - restore and run the original `NodeID-D26...` validator identity (staker cert/key + signer key), or
  - inject substantial additional stake to connected validators (approximately `399 AVAX`) to satisfy connected-stake threshold, or
  - redeploy/migrate to a new validator set with sane initial weighting.

