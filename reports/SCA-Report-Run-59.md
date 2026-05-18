# OSS Security Scan Report - Run 59

- Repository: `hbonesctr/sca-scanner-template`
- Branch/Ref: `main`
- Commit: `63ea1d95171c8e2c21acb2a68d9481d656ddbd5e`
- Generated UTC: `2026-05-18T15:00:49.843335Z`

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
