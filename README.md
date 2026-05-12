# Open Source Software (OSS) Security Scanner

**Version:** 2.4
**Author:** Hector L. Bones
**Last Updated:** 2026-05-11

A browser-based, mobile-friendly security scanning pipeline for validating open source and customer-provided software against security compliance requirements. Runs entirely inside GitHub Actions — no local tools, no CLI, no elevated privileges required.

---

## What This Does

Every time you upload a customer software ZIP file and commit it, the workflow automatically:

1. Extracts the ZIP file safely (zip-slip protected)
1. Generates a Software Bill of Materials (SBOM) using Syft
1. Runs Software Composition Analysis (SCA) using Snyk and Grype
1. Runs Static Application Security Testing (SAST) using Semgrep
1. Runs Semantic Code Analysis using CodeQL (results in Security tab)
1. Publishes findings to the Snyk web UI via `snyk monitor`
1. Consolidates all results into a structured report
1. Commits the report back to the `reports/` folder
1. Creates a GitHub Release with a downloadable audit archive

No manual steps required after committing the ZIP.

---

## Compliance Alignment

|Standard              |Controls Addressed                                                                      |
|----------------------|----------------------------------------------------------------------------------------|
|DISA ASD STIG         |APSC-DV-002560 (SAST execution), APSC-DV-003235 (dependency scanning)                  |
|NIST SP 800-53 Rev. 5 |SA-11 (Developer Testing), SA-15 (Development Process), RA-5 (Vulnerability Monitoring) |
|NTIA SBOM Requirements|Minimum elements satisfied via Syft SPDX JSON output                                   |
|DoDI 8500.01 / 8510.01|Audit trail maintained via GitHub Releases and committed reports                        |

---

## Architecture

```
customer-software/
└── your-software.zip
        │
        ▼
┌─────────────────────────┐
│  Prepare Extracted Code  │  Safely extracts ZIP → /tmp/extracted
└────────────┬────────────┘
             │ (artifact: extracted-code)
    ┌────────┼────────┬──────────────┐
    ▼        ▼        ▼              ▼
 Syft      Snyk    Grype         Semgrep
 SBOM      SCA      SCA            SAST
    │        │        │              │
    └────────┴────────┴──────────────┘
                      │
                      ▼
          ┌───────────────────────┐
          │  Consolidate Reports   │  Builds report, commits to reports/
          └───────────┬───────────┘
                      │
                      ▼
          ┌───────────────────────┐
          │  Create GitHub Release │  Zips all outputs, attaches to release
          └───────────┬───────────┘
                      │
                      ▼
          ┌───────────────────────┐
          │  CodeQL Analysis       │  Fires after release — SARIF to Security tab
          └───────────────────────┘
```

---

## Prerequisites

### Repository Secrets

Go to your repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

|Secret       |Required|Description                                                          |
|-------------|--------|---------------------------------------------------------------------|
|`SNYK_TOKEN` |Yes     |Your Snyk API token. Found at snyk.io → Account Settings → API Token |
|`SNYK_ORG_ID`|No      |Your Snyk organization ID. If omitted, Snyk uses your default org    |

### Repository Settings

Go to **Settings** → **Actions** → **General** → **Workflow permissions**

- Select **Read and write permissions**
- Click **Save**

---

## Repository Structure

After setup your repository should look like this:

```
sca-scanner-template/
├── .github/
│   └── workflows/
│       ├── scan.yml              ← Main scanning workflow (OSS Security Scanner)
│       ├── codeql.yml            ← CodeQL semantic analysis workflow
│       └── cleanup.yml           ← Reports cleanup workflow
├── cleanup-triggers/
│   └── cleanup.trigger           ← Edit this file to trigger cleanup
├── customer-software/
│   └── (place ZIP files here)
├── reports/
│   └── (auto-generated scan outputs)
├── COMPLIANCE-OVERVIEW.md
├── REFERENCES.md
└── README.md
```

---

## Initial Setup (One Time)

### Step 1 — Copy the workflow files

In your repository, create the following files using the GitHub web editor:

1. `.github/workflows/scan.yml` — main OSS Security Scanner workflow
1. `.github/workflows/codeql.yml` — CodeQL semantic analysis workflow
1. `.github/workflows/cleanup.yml` — reports cleanup workflow
1. `cleanup-triggers/cleanup.trigger` — cleanup trigger file

To create a new file in GitHub (mobile Safari):

1. Go to **Code** tab
1. Tap the **Add file** button → **Create new file**
1. Type the full path including folders (e.g. `.github/workflows/scan.yml`)
1. Paste the file content
1. Tap **Commit new file**

### Step 2 — Add your Snyk token

1. Go to [snyk.io](https://snyk.io) → Account Settings → API Token → copy it
1. In your GitHub repo go to **Settings** → **Secrets and variables** → **Actions**
1. Tap **New repository secret**
1. Name: `SNYK_TOKEN` — Value: paste your token
1. Tap **Add secret**

### Step 3 — Set repository permissions

1. Go to **Settings** → **Actions** → **General**
1. Scroll to **Workflow permissions**
1. Select **Read and write permissions**
1. Tap **Save**

---

## How to Scan Customer Software

### Step 1 — Prepare the ZIP file

Your customer ZIP file can contain:

- Source code in any language
- Package manifests (`package.json`, `requirements.txt`, `pom.xml`, etc.)
- Pre-built binaries (for unmanaged scanning)
- Any combination of the above

The ZIP file will be safely extracted and scanned. Nested ZIPs are supported.

### Step 2 — Upload the ZIP to GitHub

1. Go to your repository → **Code** tab
1. Navigate into the `customer-software/` folder
1. Tap **Add file** → **Upload files**
1. Select your ZIP file
1. Tap **Commit changes**

The workflow starts automatically the moment you commit.

### Step 3 — Monitor the scan

1. Tap the **Actions** tab
1. Tap **Open Source Software (OSS) Security Scanner** in the left list
1. Tap the latest run to see progress
1. All scan jobs run in parallel — typical total time is 4–8 minutes
1. CodeQL fires automatically after the release is created

### Step 4 — Get the results

When the workflow completes:

**GitHub Release** (primary deliverable):

1. Go to your repository home page
1. Tap **Releases** (right sidebar)
1. Find the latest `OSS Security Scan — Run #N` release
1. Tap the attached `.zip` file to download the full audit archive
1. SARIF files from CodeQL are also attached once CodeQL completes

**Security tab** (CodeQL alerts):

1. Go to your repository → **Security** tab
1. Tap **Code scanning** to see CodeQL findings

**Committed reports** (secondary, always available):

1. Go to **Code** tab → `reports/` folder
1. Open `Security-Scan-Report-Latest.md` for the summary
1. Individual scan outputs are in separate JSON and log files

---

## Output Files Reference

|File                            |Scanner |Description                                          |
|--------------------------------|--------|-----------------------------------------------------|
|`Security-Scan-Report-Latest.md`|Workflow|Executive summary with all scanner exit codes        |
|`SCA-Report-Run-N.md`           |Workflow|Same report, numbered by run for audit trail         |
|`sbom-spdx.json`                |Syft    |SPDX 2.x Software Bill of Materials                  |
|`snyk-results.json`             |Snyk    |Dependency vulnerability findings (all-projects)     |
|`snyk-unmanaged-results.json`   |Snyk    |Unmanaged/C/C++ source vulnerability findings        |
|`snyk-test.log`                 |Snyk    |Full Snyk test console output                        |
|`snyk-monitor.log`              |Snyk    |Snyk monitor console output (Snyk UI publishing)     |
|`grype-results.json`            |Grype   |Filesystem vulnerability scan results                |
|`semgrep-results.json`          |Semgrep |SAST findings from `p/ci` ruleset                    |
|`*.sarif`                       |CodeQL  |Attached to GitHub Release after CodeQL completes    |
|`manifest-candidates.txt`       |Workflow|All detected package manifests in the extracted code |
|`extracted-tree.txt`            |Workflow|Full file listing of extracted customer code         |
|`extraction-summary.json`       |Workflow|ZIP extraction metadata and error log                |

---

## How to Read the Scan Report

Open `Security-Scan-Report-Latest.md`. The Executive Summary table shows:

|Exit Code|Meaning                                                                |
|---------|-----------------------------------------------------------------------|
|`0`      |Clean — no vulnerabilities found                                       |
|`1`      |Vulnerabilities found — review the JSON output                         |
|`2`      |Error — Snyk configuration or network issue                            |
|`3`      |No supported manifest found — Snyk could not identify the project type |
|`127`    |Tool was not installed — check the install log                         |
|`not_run`|Step was skipped (e.g. unmanaged scan disabled)                        |

For Snyk findings review `snyk-results.json`. Each finding includes CVSS score, severity, affected package, and remediation path.

For Semgrep findings review `semgrep-results.json`. Each finding includes file path, line number, rule ID, and severity.

For CodeQL findings go to the **Security** tab → **Code scanning alerts** in your repository.

---

## Snyk Web UI

When `snyk monitor` succeeds (exit code `0`), your customer software appears as a project in the Snyk dashboard at [app.snyk.io](https://app.snyk.io).

If no project appears:

1. Check `snyk-monitor.log` — look for the line `Monitoring <path>`
1. Check `snyk-monitor-status.json` — review the exit codes
1. Check `manifest-candidates.txt` — if empty, Snyk found no recognized package manifests
1. Exit code `3` on monitor means no supported project type was detected. Use the unmanaged scan results in `snyk-unmanaged-results.json` instead

---

## How to Clean Up Reports

When you want to clear the `reports/` directory (e.g. between customers or before a new scan cycle):

1. Go to **Code** tab → `cleanup-triggers/` folder
1. Tap `cleanup.trigger`
1. Tap the **pencil icon** to edit
1. Change the date on the last line to today's date
1. Tap **Commit changes**

The cleanup workflow fires automatically, deletes all files in `reports/`, and resets the trigger file.

---

## Supported Languages and Package Managers

Snyk and Grype detect vulnerabilities when manifests are present. Semgrep and CodeQL scan source code regardless of language.

|Language           |Manifest Files                                              |
|-------------------|------------------------------------------------------------|
|Python             |`requirements.txt`, `Pipfile`, `pyproject.toml`, `setup.py` |
|JavaScript / Node  |`package.json`, `package-lock.json`, `yarn.lock`            |
|Java               |`pom.xml`, `build.gradle`                                   |
|.NET / C#          |`*.csproj`, `packages.config`                               |
|Go                 |`go.mod`, `go.sum`                                          |
|Ruby               |`Gemfile`, `Gemfile.lock`                                   |
|PHP                |`composer.json`, `composer.lock`                            |
|Rust               |`Cargo.toml`, `Cargo.lock`                                  |
|C / C++ (unmanaged)|Source files scanned directly by Snyk unmanaged             |

Semgrep and CodeQL scan source code regardless of language — no manifest required.
Syft generates SBOMs from any combination of the above.

---

## Free Tier Resource Usage

|Resource               |Monthly Limit|Estimated Usage per Scan|
|-----------------------|-------------|------------------------|
|GitHub Actions minutes |2,000 min    |~6–10 min               |
|Snyk open source tests |200 tests    |1–2 tests               |
|GitHub Releases        |Unlimited    |1 release               |
|GitHub artifact storage|500 MB       |~5–20 MB                |

At one scan per week (52/year), this workflow stays well within all free tier limits.

---

## Troubleshooting

**Workflow does not start after uploading ZIP**
Confirm the file is inside the `customer-software/` folder, not the root of the repository. The push trigger watches `customer-software/**` specifically.

**Release is not created**
Check the `Consolidate Reports` job. If it shows yellow or red, the `Create GitHub Release` job is skipped. Review the `Build reports folder` step log for errors.

**CodeQL does not fire**
CodeQL triggers on `workflow_run` completion of "Open Source Software (OSS) Security Scanner". Confirm the workflow name in `codeql.yml` exactly matches the `name:` field in `scan.yml`.

**Snyk shows exit code 3 for all projects**
The customer ZIP contains no recognized package manifests. Review `manifest-candidates.txt`. If empty, the software is likely C/C++ source-only — use the `snyk-unmanaged-results.json` output instead.

**Git push fails in Consolidate Reports**
Go to **Settings** → **Actions** → **General** → **Workflow permissions** and confirm **Read and write permissions** is selected.

**Syft or Grype shows NOT_EXECUTED**
The installer download from GitHub failed, likely a transient network issue. Re-run the workflow by committing any small change to `customer-software/`.

**Reports already exist from a previous scan**
New reports overwrite old ones on each run. If you want a clean slate first, use the cleanup workflow described above.

---

## Workflow Version History

|Version|Date      |Change                                                                                                                                             |
|-------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------|
|2.4    |2026-05-11|Rebranded to Open Source Software (OSS) Security Scanner. Enhanced release page with rich data-driven body. Added Author and Date to release identity. Added CodeQL workflow. Switched to softprops/action-gh-release@v2.|
|2.3    |2026-05-09|Release now fires automatically on every successful scan. Removed mobile-incompatible workflow_dispatch guard.                                     |
|2.2    |2026-05-09|Added `actions/checkout@v4` to create-release job. Fixed "not a git repository" error on `gh release create`. Corrected SNYK_ORG_ID documentation. |
|2.1    |2026-05-08|Initial dual-scanner release with Snyk, Grype, Semgrep, Syft.                                                                                     |

---

## Security Notes

- Customer ZIP files are extracted only inside the GitHub Actions runner — they are never committed to the repository
- ZIP extraction is protected against zip-slip path traversal attacks
- All scan results are committed to `reports/` and attached to releases for audit trail integrity
- The `SNYK_TOKEN` secret is never logged or exposed in workflow output
- Workflow runs with minimum required permissions (`contents: write`, `actions: read`, `security-events: write` for CodeQL)
