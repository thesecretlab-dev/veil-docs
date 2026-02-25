# EVMBench Audit Runbook

Status: Active  
Date: 2026-02-20  
Owner: Security Lead

## 1. Purpose

Run repeatable smart-contract audits with `evmbench` and archive evidence required for launch gate `G8` (Security Audit Closure).

Repository used for audits:

- `https://github.com/0x12371C/evmbench`

Local path in this workspace:

- `.cache/evmbench`

## 2. Default Scope

Primary audit target for VEIL launch:

- `companion-evm/contracts`

Optional additional scopes:

- any Solidity package containing launch-critical contracts

## 3. Bootstrap EVMBench

From `veilvm/scripts`:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-up.ps1
```

This script:

1. clones `evmbench` to `.cache/evmbench` if missing
2. prepares `backend/.env`
3. builds `evmbench/base` and `evmbench/worker` images
4. starts `backend/compose.yml`
5. waits for API readiness at `http://127.0.0.1:1337/v1/integration/frontend`

### Sigma/OpenRouter passthrough mode

If you want evmbench to run using Sigma passthrough (no local `OPENAI_API_KEY` required), run:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-up.ps1 -UseSigmaOpenRouterPassthrough
```

Then confirm:

```powershell
Invoke-RestMethod http://127.0.0.1:1337/v1/integration/frontend
```

Expected: `key_predefined: true`

## 4. Run an Audit

Set your key for the current terminal session:

```powershell
$env:OPENAI_API_KEY: <REDACTED>
```

Skip this key step when running in passthrough mode with `key_predefined: true`.

Run the default VEIL companion contract audit:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-audit.ps1 -FailOnFindings
```

Run a custom scope:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-audit.ps1 `
  -ContractsPath "..\companion-evm\contracts" `
  -Model "codex-gpt-5.2" `
  -TimeoutSeconds 3600 `
  -FailOnFindings
```

Allowed model keys in the current `evmbench` config:

- `codex-gpt-5.2`
- `codex-gpt-5.1-codex-max`

## 5. Evidence Output

Each run writes:

- `evidence-bundles/audit-closure/evmbench-<timestamp>/job-status.json`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/audit-result.json` (if job succeeded)
- `evidence-bundles/audit-closure/evmbench-<timestamp>/summary.md`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/audit-closure-template.md`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/run-metadata.json` (poll timeline + runtime metadata)
- `evidence-bundles/audit-closure/evmbench-<timestamp>/contract-file-digests.json` (SHA256/bytes/lines)
- `evidence-bundles/audit-closure/evmbench-<timestamp>/source-commentary.json`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/code-commentary.md`
- `evidence-bundles/audit-closure/evmbench-<timestamp>/audited-sources/*.sol` (exact source snapshot audited)
- `evidence-bundles/audit-closure/evmbench-<timestamp>/diagnostics/**/*` (backend/proxy/worker logs + manifest)
- pointer: `evidence-bundles/audit-closure/latest.txt`

Generate VEIL-branded HTML reports:

```powershell
Set-Location C:\Users\Josh\hypersdk\examples\veilvm\scripts
npm run audit:evmbench:report
```

HTML output:

- per-run: `evidence-bundles/audit-closure/evmbench-<timestamp>/report.html`
- archive dashboard: `evidence-bundles/audit-closure/index.html`
- latest redirect: `evidence-bundles/audit-closure/latest-report.html`
- per-run diagnostics explorer: `evidence-bundles/audit-closure/evmbench-<timestamp>/diagnostics/index.html`

## 6. G8 Closure Rule

`G8` can be set to `PASS` only when all of the following are true:

1. latest `evmbench` run is `succeeded`
2. every reported high/critical finding is either:
   - resolved with remediation diff evidence, or
   - explicitly accepted via signed risk exception
3. `audit-closure-template.md` is completed with owner and evidence links
4. closure package is linked in `VEIL_PRODUCTION_LAUNCH_CHECKLIST.md`

If findings remain unresolved and unsigned, launch state remains `NO-GO`.

## 7. Passthrough Troubleshooting (2026-02-20)

1. If worker logs show invalid model ID (`gpt-5.2-2025-12-11 is not a valid model ID`):
   - rebuild worker image with passthrough setup:
   - `powershell -NoProfile -ExecutionPolicy Bypass -File .\evmbench-up.ps1 -UseSigmaOpenRouterPassthrough`

2. If proxy logs show `invalid_prompt` / `invalid_union` on second `/v1/responses` call:
   - ensure backend image includes `oai_proxy` input normalization in:
   - `.cache/evmbench/backend/oai_proxy/routers/catch_all.py`
   - rebuild backend image:
   - `docker compose build backend`
   - `docker compose --profile proxy up -d --no-build`

3. Known-good run evidence:
   - `evidence-bundles/audit-closure/evmbench-20260220-133630/job-status.json` (`succeeded`)
