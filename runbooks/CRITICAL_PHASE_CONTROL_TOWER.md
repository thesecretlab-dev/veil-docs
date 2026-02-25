# VEIL Critical-Phase Control Tower

Status: Active  
Date: 2026-02-22  
Owner: Control-plane lane

## 1. Scope

This document is the control-plane source of truth for critical-phase operator command order, artifact requirements, and go/no-go criteria.

## 2. Command Truth State

From `examples/veilvm/scripts` command surfaces:

- `check:prelaunch` -> `node prelaunch-readiness.mjs` (`scripts/package.json`)
- `launch:rehearsal` -> `node build-launch-rehearsal-packet.mjs` (`scripts/package.json`)
- `launch:critical:strict` -> `powershell -File ./run-critical-phase-strict.ps1` (`scripts/package.json`)
- `launch:critical:dryrun` -> `powershell -File ./run-critical-phase-strict.ps1 -AllowUnsigned` (`scripts/package.json`)
- `launch:signers:init` -> `powershell -File ./init-launch-signer-policy.ps1` (`scripts/package.json`)
- `run-critical-phase-gates` -> `node run-critical-phase-gates.mjs` (direct node entrypoint, no `npm run` alias, strict signature policy by default)

## 3. Exact Operator Sequence

Run from `C:\Users\Josh\hypersdk\examples\veilvm\scripts`.

0. Initialize or refresh local signer policy from latest strict packet evidence (writes required signer addresses and min-signature policy):

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\init-launch-signer-policy.ps1
```

`run-critical-phase-strict.ps1` automatically consumes `..\secrets\launch-packet-signers.json` unless `-IgnoreSignerPolicy` is set.

1. Execute prelaunch readiness and persist an operator artifact:

```powershell
$stamp = Get-Date -Format "yyyyMMdd-HHmmss"
New-Item -ItemType Directory -Force ..\evidence-bundles\control-tower | Out-Null
npm run --silent check:prelaunch |
  Tee-Object "..\evidence-bundles\control-tower\prelaunch-readiness-$stamp.json"
```

2. Execute launch rehearsal packet generation:

```powershell
npm run launch:rehearsal
```

3. Optional consolidated orchestrator run (re-runs prelaunch + rehearsal in isolated output under `evidence-bundles/critical-phase`, strict packet signatures required):

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\run-critical-phase-strict.ps1
```

3b. Optional unsigned dry-run only (explicit non-production mode):

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\run-critical-phase-strict.ps1 -AllowUnsigned
```

4. Read rehearsal pointer:

```powershell
Get-Content ..\evidence-bundles\launch-rehearsal\latest.txt
```

## 4. Go/No-Go Criteria (Artifact-Bound)

### 4.1 Prelaunch Artifact

Required file:

- `evidence-bundles/control-tower/prelaunch-readiness-<timestamp>.json`

Go criteria:

- JSON parses successfully.
- `overallPass` is `true`.
- `checks.checklist.ok` is `true`.
- `checks.companion.ok` is `true`.
- `checks.tokenomics.ok` is `true`.
- `checks.auditClosure.ok` is `true`.
- `checks.flywheelAudit.ok` is `true`.
- `checks.economicCoherence.ok` is `true`.
- `checks.animaReadiness.ok` is `true`.

No-go criteria:

- File missing or malformed JSON.
- Any required boolean above is `false`.

### 4.2 Rehearsal Artifacts

Pointer file:

- `evidence-bundles/launch-rehearsal/latest.txt`

Required files under pointed run directory:

- `rehearsal-report.json`
- `rehearsal-report.md`
- `launch-packet.json`
- `launch-packet.md`
- `rollback-runbook.md`

Go criteria:

- `rehearsal-report.json.overallPass` is `true`.
- Every `rehearsal-report.json.checks[]` row with `required=true` has `ok=true`.
- `launch-packet.json.signatureCheckPass` is `true`.
- `launch-packet.json.allowUnsigned` is `false`.
- `launch-packet.json.validSignatures.length >= launch-packet.json.minSignatures`.
- `launch-packet.json.missingRequiredSigners.length` is `0`.

No-go criteria:

- Pointer missing or points to missing run directory.
- Any required file missing.
- Any go criterion above fails.

### 4.3 Optional Critical-Phase Summary Artifacts

Pointer file (only if step 3 was run):

- `evidence-bundles/critical-phase/latest.txt`

Required files under pointed run directory:

- `critical-phase-summary.json`
- `critical-phase-summary.md`

Go criteria:

- `critical-phase-summary.json.overallPass` is `true`.
- `critical-phase-summary.json.requiredFailedChecks.length` is `0`.
- `critical-phase-summary.json.stages.prelaunchReadiness.reportOverallPass` is `true`.
- `critical-phase-summary.json.stages.launchRehearsalPacket.reportOverallPass` is `true`.
- `critical-phase-summary.json.stages.launchRehearsalPacket.productionEligible` is `true`.
- For `critical-phase-summary.json.stages.launchRehearsalPacket.rehearsalRunDir`, the linked `launch-packet.json.allowUnsigned` is `false`.

No-go criteria:

- Pointer missing or points to missing run directory.
- Any required file missing.
- Any go criterion above fails.

## 5. Signature Requirements

Launch packet signature policy is enforced by `scripts/build-launch-rehearsal-packet.mjs` (`LH11_PACKET_SIGNATURES`).

Inputs:

- `VEIL_LAUNCH_PACKET_SIGNER_KEYS` or `--signer-private-keys`
- `VEIL_LAUNCH_PACKET_REQUIRED_SIGNERS` or `--required-signers`
- `VEIL_LAUNCH_PACKET_SIGNATURES` or `--external-signatures`
- `VEIL_LAUNCH_PACKET_MIN_SIGNATURES` or `--min-signatures` (default `1`)
- local strict-run key file: `examples/veilvm/secrets/launch-packet-signer.pk`
- local signer policy file: `examples/veilvm/secrets/launch-packet-signers.json` (created by `scripts/init-launch-signer-policy.ps1`)
- policy template: `scripts/launch-packet-signers.template.json`

Signature pass condition when unsigned bypass is off:

- Valid signatures count is at least `minSignatures`.
- All required signer addresses are present in valid signatures.

## 6. Unsigned Dry-Run Caveat

Unsigned bypass is enabled by either:

- `--allow-unsigned`
- `VEIL_LAUNCH_PACKET_ALLOW_UNSIGNED=true`

This bypass marks signature policy as pass for rehearsal mechanics only.  
Any packet with `launch-packet.json.allowUnsigned=true` is automatic **NO-GO** for critical-phase launch authorization.
