# Security Scan Report - Run 53

- Repository: `hbonesctr/sca-scanner-template`
- Branch/Ref: `main`
- Commit: `e5b34b186353ad4d8de64200ccf57c67748582e6`
- Generated UTC: `2026-05-09T19:24:05.749880Z`

## Executive Summary

| Component | Status | Evidence |
|---|---:|---|
| Extraction | PRESENT | `reports/extraction-summary.json` |
| Manifest Candidates | 5 found | `reports/manifest-candidates.txt` |
| Syft SBOM | PRESENT | `reports/sbom-spdx.json` |
| Snyk Test | all-projects exit `2`, unmanaged exit `147` | `reports/snyk-results.json`, `reports/snyk-unmanaged-results.json` |
| Snyk Monitor | all-projects exit `2`, unmanaged exit `0` | `reports/snyk-monitor.log`, `reports/snyk-unmanaged-monitor.log` |
| Grype | scan exit `0` | `reports/grype-results.json` |
| Semgrep | scan exit `0` | `reports/semgrep-results.json` |

## ZIP Extraction

- ZIP files found: `1`
- ZIP files extracted: `1`
- Non-ZIP content copied: `True`

## Snyk UI Notes

- For Snyk projects to appear in the Snyk web UI, `snyk monitor` must complete successfully.
- If `snyk monitor --all-projects` returns exit code `3`, Snyk did not detect supported package manifests.
- This workflow also runs `snyk monitor --unmanaged` when enabled, which is useful for C/C++ or source-only archives.
- Confirm that GitHub repository secrets `SNYK_TOKEN` and optionally `SNYK_ORG_ID` are configured.

## Key Files

- `reports/manifest-candidates.txt`
- `reports/snyk-results.json`
- `reports/snyk-unmanaged-results.json`
- `reports/snyk-monitor.log`
- `reports/snyk-unmanaged-monitor.log`
- `reports/grype-results.json`
- `reports/semgrep-results.json`
- `reports/sbom-spdx.json`
