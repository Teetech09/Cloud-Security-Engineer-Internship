Phase A: Theory & Introduction

Objective

The objective of this phase was to understand how secrets become exposed in source code and what that exposure can mean for cloud infrastructure. It also introduced TruffleHog as a tool for detecting exposed credentials, using a deliberately seeded demo repository.

Actions Taken

I began by reviewing the common causes of secret exposure: a missing or misconfigured .gitignore that lets files such as .env or key files be committed, developer oversight such as hardcoding credentials for testing, secrets that remain in git history after the file is deleted, and CI/CD mismanagement such as tokens printed in build logs or stored as plaintext pipeline variables. I then looked at how attackers use automated scanners and code search against public repositories to harvest credentials, often within minutes of a push.

For the practical part, I installed TruffleHog version 3.99.7 inside WSL2 Ubuntu on my Windows 10 laptop. I cloned the demo repository with `git clone https://github.com/trufflesecurity/test_keys.git` and scanned it with `trufflehog filesystem test_keys`. My first scan returned nothing because I had run it before the clone completed, so the folder did not exist yet. After cloning the repository and confirming its contents with `ls`, the scan ran correctly.

Key Observations

The scan produced two findings. The first was a verified AWS access key in `test_keys/new_key` at line 2, which TruffleHog confirmed against AWS. The tool reported that this key is a canary token from canarytokens.org, meaning it is a decoy that alerts its owner when used, which made it safe to scan and document. The second was an unverified OpenSSH private key, which TruffleHog flagged as a possible secret but could not test against a service.

The output format shows the detector type, decoder type, the raw result, the file and the line number, along with a verification status. TruffleHog version 3 relies on detectors and live verification rather than entropy scores, which helps separate confirmed exposures from possible false positives and tells a responder exactly where to remediate. The scan covered the working tree only, so a separate scan of the git history would be needed to catch secrets that were deleted in earlier commits.

If a verified key like this belonged to a real IAM user, an attacker could use it to access whatever that user's permissions allow, such as reading S3 data, launching compute resources or enumerating IAM, and could use it to move laterally across the account. The right response is to revoke and rotate the key immediately, replace long-lived keys with temporary credentials or Secrets Manager, and add pre-commit scanning and a strict .gitignore to prevent a repeat.

Evidence Produced

The evidence file is `phaseA_scan.png`, a screenshot of the TruffleHog output highlighting the verified AWS access key detected in `test_keys/new_key` at line 2.
