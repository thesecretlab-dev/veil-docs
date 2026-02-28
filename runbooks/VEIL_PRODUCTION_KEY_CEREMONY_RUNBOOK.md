# VEIL Production Key Ceremony Runbook

## Scope

This runbook covers production execution of VeilVM threshold-keying ceremony for encrypted mempool tx gossip.

Repository scope:

- `<local-dev-path>`

This runbook is Docker-first to avoid local CGO/build issues (`DataDog/zstd`, `blst`) on operator machines.

## Outputs

A successful ceremony produces:

- `evidence-bundles/key-ceremony/ceremony-<timestamp>-prod/ceremony-manifest.json`
- `evidence-bundles/key-ceremony/ceremony-<timestamp>-prod/committee-public-keys.json`
- `evidence-bundles/key-ceremony/ceremony-<timestamp>-prod/node-public/*.public.json`

Optional promotion:

- `evidence-bundles/key-ceremony/latest/*`
- `evidence-bundles/key-ceremony/latest.txt`

## Roles

- `Ceremony Lead`: assembles committee map and runs ceremony.
- `Validator Operator`: generates and protects local private key <REDACTED> then builds local runtime env.
- `Security Reviewer`: verifies manifest digest, committee membership, and artifact integrity.

## Preconditions

1. Final validator committee roster is frozen (`NodeID-*` list).
2. `MinShares` is approved (must be `>= 2` and `<= committee size`).
3. Docker is installed and running on ceremony host.
4. `veilvm-node:latest` image is available on ceremony host.
5. Every validator operator has generated and retained a local threshold private key.

## Step 1: Generate Per-Validator Public Records

Run on each validator operator host.

```powershell
Set-Location <local-dev-path>

$nodeID = "NodeID-REPLACE_ME"
$outDir = ".\evidence-bundles\key-ceremony\node-keys\$nodeID"

$mount = "<local-dev-path>"
docker run --rm `
  -v $mount `
  -w /work `
  veilvm-node:latest `
  veilvm-keygen `
  --mode threshold-node `
  --node-id $nodeID `
  --out "/work/evidence-bundles/key-ceremony/node-keys/$nodeID"
```

Expected files for each validator:

- `<NodeID>.public.json` (share with ceremony lead)
- `<NodeID>.private.env` (keep private; do not share)

## Step 2: Run Production Ceremony (Ceremony Lead)

Collect all validator `*.public.json` files into one directory and run:

```powershell
Set-Location <local-dev-path>

powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run-production-key-ceremony.ps1 `
  -CommitteeNodeIDs "NodeID-A,NodeID-B,NodeID-C" `
  -PublicRecordsDir ".\evidence-bundles\key-ceremony\node-keys" `
  -MinShares 2 `
  -PromoteLatest
```

Notes:

- Script runs `veilvm-keygen` in Docker (not local `go run`).
- Script intentionally uses `--emit-node-env=false` to avoid centralizing private keys.

## Step 3: Build Per-Validator Runtime Env (Operator Side)

Run on each validator host with local private env file:

```powershell
Set-Location <local-dev-path>

powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\build-threshold-node-env.ps1 `
  -NodeID "NodeID-A" `
  -CeremonyDir ".\evidence-bundles\key-ceremony\latest" `
  -PrivateEnvPath ".\evidence-bundles\key-ceremony\node-keys\NodeID-A\NodeID-A.private.env" `
  -OutFile ".\secrets\threshold\NodeID-A.env"
```

The generated env file contains:

- `VEIL_TX_GOSSIP_ENCRYPTION_REQUIRED=true`
- `VEIL_TX_GOSSIP_THRESHOLD_DECRYPT_ENABLED=true`
- `VEIL_TX_GOSSIP_THRESHOLD_MIN_SHARES=<N>`
- `VEIL_TX_GOSSIP_THRESHOLD_NODE_PRIVATE_KEY_HEX=<local-private>`
- `VEIL_TX_GOSSIP_THRESHOLD_COMMITTEE_PUBLIC_KEYS_JSON=<committee-map>`

## Step 4: Rollout + Verification

1. Deploy validator env updates.
2. Restart validator processes in rollout order.
3. Confirm runtime marker appears per validator:
   `cryptographic threshold tx gossip keying enabled`
4. Run rollout audit:

```powershell
Set-Location <local-dev-path>

node .\scripts\audit-threshold-keying-rollout.mjs `
  --manifest .\evidence-bundles\key-ceremony\latest\ceremony-manifest.json `
  --rollout .\scripts\threshold-keying.rollout.json `
  --log-tail 25000
```

Audit must end with:

- `"overallPass": true`

## Step 5: Evidence Archive Requirements

Archive all of:

1. Ceremony manifest and committee map.
2. Committee digest approval record.
3. Rollout audit JSON/Markdown bundle.
4. Validator runtime marker log excerpts.
5. Custody attestations for private key <REDACTED>

## Hard Rules

1. Never commit private key <REDACTED> (`*.private.env`) to git.
2. Never centralize validator private threshold keys with ceremony lead.
3. Never mark production privacy as complete without passing rollout evidence and marker verification across the full committee.

