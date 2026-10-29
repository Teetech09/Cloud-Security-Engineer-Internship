Phase C: Core Execution, Secret Detection & Validation

Objective

The objective of this phase was to detect exposed secrets in a repository and validate whether a detected credential is still live against the sandboxed AWS environment. The phase connects detection, which finds a candidate secret, with validation, which shows whether it is an active exposure that needs an incident response.

 Actions Taken

The task brief pointed to a demo repository named `pwnedlabs/leaked-creds-demo`, but I could not find it, so I built an equivalent target myself. In WSL2 Ubuntu I created a local git repository and committed a `.env` file containing the access key pair for the CloudGoat `manager` user from Phase B, then removed the file in a second commit. This reproduces a common real-world mistake: a developer deletes a file, but the credential stays in the commit history. I kept this repository local and did not push it to GitHub, because its history contains a live key.

I scanned the repository with TruffleHog 3.99.7 using `trufflehog git file://$(pwd)` and with Gitleaks 8.30.1 using `gitleaks detect --source . --report-format json --report-path gitleaks_report.json`. I installed Gitleaks from its release archive and found that this build uses `detect` rather than the newer `git` command. I then parsed the Gitleaks JSON report with jq to extract the rule, file, line, commit and a truncated secret value for each finding, which avoids printing the full key.

To validate the finding, I recovered the key pair from the repository's history with `git show`, configured it under a separate AWS CLI profile named `test_leak` and ran `aws sts get-caller-identity --profile test_leak`. I only validated a key that belongs to my own sandbox, because testing credentials found in someone else's repository would be unauthorized access.

Observation

Both tools detected the exposure, but they did so differently. TruffleHog reported one verified secret and no unverified ones, because it tests a candidate key against AWS, while Gitleaks reported one leak in the initial commit by matching patterns without checking whether the key works. Together they show why a verified result is the higher priority: it is a confirmed, active credential, whereas a pattern match still needs triage. The Gitleaks report also showed that the finding sat in the first commit, not in the final state of the repository.

The validation step returned the identity of the `manager` IAM user, which confirmed that the key still authenticated even though the `.env` file had been deleted. This is the central lesson of the phase: removing a file does not revoke a credential, and anyone with access to the history can recover it. I therefore classified the key as compromised, and the correct response is to rotate it and review its activity, which is the focus of Phase D. Longer term, the controls that would have prevented this are a strict .gitignore, pre-commit and CI secret scanning, and temporary credentials or a managed secrets service in place of long-lived keys.

Evidence Produced
*PhaseC_validation.png: Screenshot of `aws sts get-caller-identity --profile test_leak` returning the identity of the IAM user `manager_cgidndh53tyxvd`, with the account and user IDs masked.
*PhaseC_jq_extract.png: Screenshot of the jq extraction from the Gitleaks report, showing the rule `aws-access-token`, the file `.env`, the line number and commit, with the secret truncated.
* PhaseC_findings.txt: Summary of the scan results, the validation outcome, and the compromised status of the key.
* PhaseC_gitleaks_report.json: Gitleaks JSON report for the demo repository, with the secret and match values redacted before publishing.
