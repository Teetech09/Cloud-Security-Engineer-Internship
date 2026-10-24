 Phase C: Core Execution — Secret Discovery

 Objective
Identify potential exposed secrets in a sample repository using TruffleHog and Gitleaks, then assess the findings in context.

 Tools and Target
- Ubuntu on WSL2
- TruffleHog 3.99.7
- Gitleaks installed through Ubuntu’s package manager
- Target repository: danielmiessler/SecLists

Actions Taken

1. Prepared the Repository
Cloned SecLists into my lab directory:

```bash
mkdir -p ~/secret-discovery-lab
cd ~/secret-discovery-lab
git clone https://github.com/danielmiessler/SecLists.git
```

Checked the repository status:

```bash
git -C SecLists status --short --branch
```

The output showed a clean checkout on `master`, tracking `origin/master`.

 2. Ran TruffleHog
The initial scan failed because TruffleHog’s temporary repository copy exceeded the available space in `/tmp`.

Disk checks showed that `/tmp` had a 1.9 GB limit, while the Ubuntu filesystem had sufficient free space. I created a temporary directory under my home folder and reran the scan:

```bash
mkdir -p "$HOME/secret-discovery-lab/tmp"

TMPDIR="$HOME/secret-discovery-lab/tmp" trufflehog git \
  "file://$HOME/secret-discovery-lab/SecLists" \
  --no-verification --no-update > trufflehog_output.txt
```

TruffleHog reported that scanning finished.

3. Attempted Gitleaks
Started a Git-history scan:

```bash
gitleaks detect --source ./SecLists \
  --report-format json \
  --report-path gitleaks_report.json \
  --redact
```

I interrupted this scan after a prolonged runtime. It is not recorded as complete.

To reduce the workload, I switched to scanning the current files:

```bash
gitleaks detect --source ./SecLists --no-git \
  --report-format json \
  --report-path gitleaks_report.json \
  --redact
```

Completion of this second scan has not yet been confirmed.

## TruffleHog Results

| Metric | Reported result |
|---|---|
| Scan duration | 27 minutes, 33 seconds |
| Chunks processed | 1,956,682 |
| Bytes processed | 7,952,317,129 |
| Unverified findings | 13,519 |
| Verified secrets | 0 |
| Output file | trufflehog_output.txt |

Coverage Limitation
TruffleHog reported a processing timeout for:

`Payloads/Zip-Bombs/42-passwordless.zip`

The scanner reached its completion message, but this error limits coverage of that archive.

 Key Observations
- Scanning a large repository’s Git history can require substantial time, memory and temporary storage.
- A temporary filesystem can run out of space even when the main filesystem has sufficient capacity.
- SecLists contains intentional security testing data. A detected pattern does not, by itself, establish an accidental credential exposure.
- TruffleHog’s 13,519 findings are unverified detections, not necessarily unique credentials or confirmed leaks.
- Verification was explicitly disabled. The reported zero verified secrets therefore does not establish that all findings are harmless.
- TruffleHog scanned Git history, while the revised Gitleaks command targets current files. Their finding counts cannot be compared directly without accounting for this difference.

Preliminary Analysis
The TruffleHog output requires contextual review before findings can be classified as test data, false positives or potentially sensitive exposures. Credential-like values should be assessed using their file location, surrounding content and intended purpose.

No finding has yet been established as a valid, active credential leak. A final comparison remains pending until Gitleaks completes and representative findings from both tools are reviewed.

Evidence Status

| Evidence | Status |
|---|---|
| trufflehog_output.txt | Generated; approximately 5 MB |
| gitleaks_report.json | Completion and report validity not confirmed |
| phaseC_summary.png | Pending final analysis |

Raw reports must be reviewed and sensitive values redacted before publication.

 Completion Checklist
- [x] TruffleHog reached its completion message; archive timeout documented.
- [ ] Gitleaks scan completed and JSON report validated.
- [ ] Findings reviewed and compared.
- [ ] Final AI summary captured as phaseC_summary.png.
