# VEIL Mempool Privacy Hardening Runbook

Status: Active  
Date: 2026-02-21  
Owner: VM Lead + Security Lead

## 1. Purpose

Document the mempool privacy hardening step that is now implemented, what it guarantees, and what is still required before claiming fully private mempool behavior.

## 2. Implemented in this Step

### 2.1 Encrypted tx gossip transport

- Tx gossip payloads are no longer sent as plaintext batched tx bytes.
- Payloads are wrapped in authenticated encryption (`AES-256-GCM`) before p2p relay.
- Implementation:
  - `chain/transaction_marshaller_encrypted.go`
  - `vm/vm.go`

### 2.2 Fail-closed runtime wiring

- VM startup now fails if gossip encryption is required and no valid key is configured.
- VM config and env support:
  - `txGossipEncryptionKeyHex`
  - `txGossipEncryptionRequired`
  - env overrides: `VEIL_TX_GOSSIP_ENCRYPTION_*`, `HYPERSDK_TX_GOSSIP_ENCRYPTION_*`
- Implementation:
  - `vm/config.go`
  - `vm/vm.go`

### 2.3 Secret hygiene hardening

- Local docker profile no longer carries a repo-embedded default gossip key.
- Node now reads secret from local git-ignored `examples/veilvm/.env`.
- Implementation:
  - `examples/veilvm/docker-compose.local.yml`

### 2.4 Test coverage

- Added serializer tests for:
  - invalid key rejection
  - encrypted roundtrip
  - wrong-key decrypt rejection
  - plaintext payload rejection
- Test file:
  - `chain/transaction_marshaller_encrypted_test.go`

### 2.5 Threshold-gated decryption release

- Added threshold envelope framing for tx gossip and a quorum gate before release into mempool submit path.
- Encrypted gossip payloads are held until at least `txGossipThresholdMinShares` unique validator attestations are observed.
- Runtime fail-close rules:
  - threshold mode requires encrypted gossip mode
  - minimum shares must be `>=2`
- Implementation:
  - `chain/transaction_marshaller_threshold.go`
  - `vm/threshold_tx_gossip_handler.go`
  - `vm/config.go`
  - `vm/vm.go`
- Test files:
  - `chain/transaction_marshaller_threshold_test.go`
  - `vm/threshold_tx_gossip_handler_test.go`

### 2.6 Cryptographic threshold keying mode (implemented)

- Added optional cryptographic threshold keying path:
  - per-envelope data key is Shamir-split
  - each share is X25519-encrypted to a committee validator key
  - threshold shares are combined before decrypting envelope payload into mempool submit path
- Runtime activation requires:
  - `txGossipThresholdNodePrivateKeyHex`
  - `txGossipThresholdCommitteePublicKeys` (or env JSON equivalent)
- Implementation:
  - `vm/threshold_tx_gossip_crypto.go`
  - `vm/threshold_tx_gossip_crypto_handler.go`
- Test file:
  - `vm/threshold_tx_gossip_crypto_test.go`

## 3. What This Guarantees Now

- Tx gossip is confidential and tamper-evident on the network transport path.
- Nodes cannot silently run plaintext gossip when encryption is marked required.
- Local key is managed as a secret file, not committed config.
- In threshold mode, encrypted gossip is not released to mempool submit path until validator-share quorum is reached.
- In cryptographic threshold mode, no single validator local key share can directly decrypt envelope data key before threshold combine.

## 4. What This Does Not Yet Guarantee

- Cryptographic threshold keying code path is implemented but requires production key ceremony and signer mapping rollout before launch claims.
- Full production guarantee still requires adversarial evidence that runtime config is threshold-keying mode (not fallback/attestation mode) across validator set.

## 5. Verification Procedure

Run from repo root (`C:\Users\Josh\hypersdk`) unless noted.

1. Build and restart node with hardened compose profile:

```powershell
docker build -t veilvm-node:latest -f examples/veilvm/Dockerfile.node .
cd examples/veilvm
docker compose -f docker-compose.local.yml up -d --force-recreate node
```

2. Verify required encryption env is present in running container:

```powershell
docker exec veilvm-node /bin/sh -lc 'echo VEIL_TX_GOSSIP_ENCRYPTION_REQUIRED=$VEIL_TX_GOSSIP_ENCRYPTION_REQUIRED; echo VEIL_TX_GOSSIP_ENCRYPTION_KEY_HEX=${VEIL_TX_GOSSIP_ENCRYPTION_KEY_HEX:+set}'
```

Expected:

- `VEIL_TX_GOSSIP_ENCRYPTION_REQUIRED=true`
- `VEIL_TX_GOSSIP_ENCRYPTION_KEY_HEX=set`

3. Verify chain readiness:

```powershell
Invoke-WebRequest -UseBasicParsing http://127.0.0.1:9660/ext/health/readiness
```

Expected: `healthy: true`

4. Verify serializer + threshold-gossip + cryptographic threshold-keying tests in Docker toolchain:

```powershell
docker run --rm -v ${PWD}:/src -w /src golang:1.23-bookworm bash -lc '/usr/local/go/bin/go test ./chain -run "Threshold|EncryptedBatchedTransactionSerializer" -count=1 && /usr/local/go/bin/go test ./vm -run "ThresholdTxGossip" -count=1'
```

Expected:

- `ok github.com/ava-labs/hypersdk/chain`
- `ok github.com/ava-labs/hypersdk/vm`

## 6. Launch Gate Mapping

- Gate: `G2 Native Privacy Invariants`
- Current posture:
  - PASS for encrypted gossip transport + fail-close wiring in local profile.
  - PASS (local) for threshold-gated mempool decrypt release path.
  - IN PROGRESS for production cryptographic threshold-keying rollout (key ceremony, config activation, adversarial replay evidence).

## 7. Next Mandatory Step (for fully private mempool)

Activate cryptographic threshold decryption in production validator profile:

- encrypted envelopes remain unreadable until threshold share condition is met
- no single validator can decrypt mempool payload unilaterally
- missing/invalid share path fails closed with deterministic rejection

Acceptance criteria:

1. Single-node key compromise cannot decrypt gossip payloads.
2. Threshold share flow is deterministic and consensus-safe.
3. Mixed-key/missing-key/replay/tamper test matrix passes and archived.
4. Launch docs and evidence bundles updated with pass/fail artifacts.

Execution procedure (key ceremony + rollout artifact generation):

1. Each validator generates local threshold key material (private output stays local):

```powershell
go run .\examples\veilvm\cmd\veilvm-keygen --mode threshold-node --node-id <NodeID-...> --out .\examples\veilvm\evidence-bundles\key-ceremony\node-keys
```

2. Security lead assembles committee map + manifest:

```powershell
go run .\examples\veilvm\cmd\veilvm-keygen `
  --mode threshold-ceremony `
  --committee-node-ids <NodeID-A>,<NodeID-B>,<NodeID-C>,<NodeID-D> `
  --min-shares 3 `
  --committee-public-keys-file .\examples\veilvm\evidence-bundles\key-ceremony\committee-public-input.json `
  --committee-private-keys-file .\examples\veilvm\evidence-bundles\key-ceremony\committee-private-input.json `
  --out .\examples\veilvm\evidence-bundles\key-ceremony\ceremony-<timestamp>
```

Security note: `committee-private-input.json` and `node-env/*.env` contain secret key: <REDACTED> do not commit them and keep them in encrypted operator storage.

3. Distribute per-node env snippets (`node-env/<NodeID>.env`) and restart validators.
   - Local docker path: `docker-compose.local.yml` now materializes per-chain VM `config.json` from `.env` before AvalancheGo starts.
   - This is required because plugin subprocesses only receive runtime engine env vars.

4. Archive the ceremony bundle in launch evidence:
   - `committee-public-keys.json`
   - `ceremony-manifest.json` (includes digest + minShares + source attribution)
   - rollout proof (validator config hash + startup logs showing `cryptographic threshold tx gossip keying enabled`)

5. Run deterministic rollout activation audit and archive bundle:

```powershell
cd .\examples\veilvm\scripts
Copy-Item .\threshold-keying.rollout.template.json .\threshold-keying.rollout.json
# fill validator entries in threshold-keying.rollout.json
npm run audit:threshold-keying -- `
  --manifest ..\evidence-bundles\key-ceremony\ceremony-<timestamp>\ceremony-manifest.json `
  --rollout .\threshold-keying.rollout.json `
  --log-tail 2500
```

Expected outputs:

- `examples/veilvm/evidence-bundles/threshold-keying-rollout/tkroll-*/threshold-keying-rollout.json`
- `examples/veilvm/evidence-bundles/threshold-keying-rollout/tkroll-*/threshold-keying-rollout.md`
- `examples/veilvm/evidence-bundles/threshold-keying-rollout/latest.txt`

Current local snapshot (`2026-02-22`, current VeilVM stack only):

- latest run: `examples/veilvm/evidence-bundles/threshold-keying-rollout/tkroll-20260222-190103`
- result: `overall_pass=true`
- validator coverage:
  - primary validator (`NodeID-BRWmyj4aQPgx1suA3Le9km1aF6sQnmVyw`) PASS
  - secondary validator (`NodeID-AvwgLY7XaaisHondANg4MTB58WV5StVgD`) PASS
- runtime marker checks:
  - required marker present on both validators: `cryptographic threshold tx gossip keying enabled`
  - forbidden fallback marker absent on both validators: `threshold-gated tx gossip decryption enabled (attestation mode)`
- audit script note:
  - rollout audit now reads docker logs from container start time (or explicit `logSince`) to avoid startup marker false negatives from short tail windows
