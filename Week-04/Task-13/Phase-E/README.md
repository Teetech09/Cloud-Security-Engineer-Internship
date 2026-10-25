 Phase E: Hosted Labs Integration — Secret Exposure Detection

Objective
Reinforce secret-detection skills using external labs, applying the same techniques practiced in Phase C (TruffleHog, GitLeaks) against realistic, hosted scenarios: secrets left in git history, injected into Docker images, and buried in a CodeCommit repository.

Tools Used
- `git-secrets` (AWS Labs) — pre-commit secret scanning and history scanning
- `git log` / `git show` — manual git history inspection
- AWS CLI — S3, STS, CodeCommit
- Docker Desktop (WSL2 backend)
- `aws-enumerator` (Go tool) — AWS permission enumeration

---

 Exercise 0: git-secrets Setup

Before starting the hosted labs, `git-secrets` was installed and tested locally to understand how pre-commit secret scanning works in practice.

Steps:
1. Cloned and installed `git-secrets` on WSL2 Ubuntu (`sudo make install`).
2. Enabled hooks in a test repo: `git secrets --install`, `git secrets --register-aws`.
3. Attempted to commit a fake AWS key (`AKIAIOSFODNN7ABCDEFG`) — the commit was **rejected** by the pre-commit hook, with the matched pattern and line shown.
4. Bypassed the hook with `--no-verify`, then ran `git secrets --scan-history`, which still caught the key in the commit history — demonstrating that hooks alone don't guarantee a clean history if bypassed.

Key takeaway: `git-secrets` is a *prevention* tool (stops a secret before it's committed); tools like TruffleHog/GitLeaks are *detection* tools (find secrets already committed). A solid pipeline uses both — prevention at commit time, detection in CI as a backstop.

---

 Lab 1: Hunt for Secrets in Git Repos (PwnedLabs)

Target: `huge-logistics/cargo-logistics-dev` — a logistics company's public GitHub repo.

 Steps
1. **Cloned the repo** and ran `git secrets --scan`, which found nothing in the current working tree.
2. **Ran `git secrets --scan-history`**, which flagged an AWS access key inside a previous commit, in `log-s3-test/log-upload.php`.
3. **Inspected the deleted file directly from the commit** that removed it:
   ```
   git show <commit-id>:log-s3-test/log-upload.php
   ```
   This revealed a hardcoded AWS access key and secret key, plus a reference to an S3 bucket (`huge-logistics-transact`).
4. Cross-checked manually for secrets automated tools might miss, using `grep -R "mysqli_connect"`, which surfaced a hardcoded database password — a reminder that scanners catch known patterns, not every kind of secret.
5. Validated the leaked AWS key in an isolated CLI profile:
   ```
   aws sts get-caller-identity --profile lab1
   ```
   The key was live.
6. Enumerated and accessed the bucket:
   ```
   aws s3 ls s3://huge-logistics-transact --profile lab1
   ```
   Found `flag.txt`, `transact.log`, and `web_transactions.csv` (the latter containing PII — usernames, emails, IP addresses).
   Retrieved the flag from `flag.txt` and submitted it to complete the lab.

 Finding
A deleted file containing a hardcoded AWS key remained fully recoverable from git history, days after removal. The key was still active and granted read access to a bucket holding customer PII.

 Remediation
- Never hardcode credentials in source files — use environment variables injected at deploy time, or a secrets manager (AWS Secrets Manager, SSM Parameter Store).
- Deleting a file from the latest commit does **not** remove it from history — a full history rewrite (`git filter-repo` / BFG) and key rotation are both required once a secret is exposed.
- Enable `git-secrets` or an equivalent pre-commit hook repo-wide to catch this before it's ever committed.
- Rotate any credential immediately once it's known to have been exposed, regardless of how long ago.

---

Lab 2: Uncover Secrets in CodeCommit and Docker (PwnedLabs)

Target: A public Docker Hub image, `hljose/huge-logistics-terraform-runner:0.12`, and a private AWS CodeCommit repository reachable through credentials found inside it.

Steps
1. Located the image on Docker Hub via a name-based search for the target company.
2. Pulled and inspected the image. Note: local `docker pull` was blocked for most of this session by a Docker Desktop WSL2 networking issue (intermittent DNS/TLS failures reaching `docker.io` and related CDN endpoints — tried IPv4/IPv6 adjustments, mirrored WSL networking, host networking, and a full Docker Desktop reset, without a reliable fix). The container's environment variables were instead taken from the lab's own documented walkthrough, which shows the same `docker run ... env` output.
3. Found AWS keys injected as container environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) — a common but risky practice for "convenient" image distribution.
4. Confirmed the identity:
   ```
   aws sts get-caller-identity --profile lab2
   ```
   Principal: `prod-deploy`.
5. Attempted automated permission enumeration with `aws-enumerator`. The `enum` step ran, but `dump` crashed (a bug in this unmaintained 2022 tool when the result set is empty). Fell back to direct AWS CLI calls instead, which achieved the same goal.
6. Enumerated CodeCommit directly:
   ```
   aws codecommit list-repositories --profile lab2
   ```
   Found a repository: `vessel-tracking`.
7. Walked the commit history on the `dev` branch:
   ```
   aws codecommit get-branch --repository-name vessel-tracking --branch-name dev --profile lab2
   aws codecommit get-commit --repository-name vessel-tracking --commit-id <commit-id> --profile lab2
   aws codecommit get-differences --repository-name vessel-tracking --before-commit-specifier <parent-id> --after-commit-specifier <commit-id> --profile lab2
   ```
   This identified `js/server.js` as a file changed between two commits.
8. Retrieved and decoded the file:
   ```
   aws codecommit get-file --repository-name vessel-tracking --commit-specifier <commit-id> --file-path js/server.js --profile lab2
   ```
   The response's base64 `fileContent` field, decoded, revealed a **second** set of hardcoded AWS keys directly in the application source, plus the S3 bucket name `vessel-tracking`.
9. Accessed the bucket with the new keys:
   ```
   aws s3 ls s3://vessel-tracking --profile lab2b
   ```
   Found `flag.txt` alongside a large number of vessel-location data files.
10. Retrieved the flag and submitted it to complete the lab.

 Finding
Two independent and reinforcing secret exposures: credentials baked into a public Docker image's environment variables, and a *second* set of credentials hardcoded in application source committed to a private CodeCommit repo. Access to either led to the same S3 bucket, holding both operational data and the flag. Compromising a container image gave a path into source control, which gave a path into cloud storage — a realistic lateral-movement chain from a single public artifact.

Remediation
- Never bake credentials into a Docker image, even "for convenience" — use IAM roles for compute, or inject secrets at runtime via a secrets manager.
- Scan container images for secrets as part of the build pipeline (`trufflehog docker --image ...`), noting that this lab's own walkthrough found it misses env-var-injected secrets, so this check alone is not sufficient — scan for env/config exposure separately.
- Apply the same source-scanning discipline (`git-secrets`, TruffleHog, GitLeaks) to internal/private repos as public ones; CodeCommit being "private" did not prevent compromise once an attacker had a foothold.
- Principle of least privilege: the `prod-deploy` principal's broad read access to CodeCommit and S3 meant one leaked image credential cascaded into two further compromises.



 Environment Notes
- All labs run on WSL2 Ubuntu (plus some AWS CLI steps from Windows PowerShell).
- Spent significant time resolving a Docker Desktop WSL2 networking issue (DNS/TLS failures against `docker.io`, `auth.docker.io`, and CloudFront endpoints). Tried: DNS troubleshooting, WSL `generateResolvConf` override, mirrored WSL networking mode, Docker Desktop host networking, IPv6 disabling, and a Docker Desktop data reset — none fully resolved it. Proceeded using the lab's documented container output instead of a live pull, which did not affect the CodeCommit/S3 portion of the lab, completed entirely with live AWS CLI calls.
- Also hit and worked around an unrelated crash in `aws-enumerator` (`dump` panics on an empty result set) by using direct `aws codecommit` / `aws s3` CLI calls instead.

 Evidence
- `phaseE_lab1.png` — PwnedLabs Lab 1 completion
- `phaseE_lab2.png` — PwnedLabs Lab 2 completion
- `phaseE_gitsecrets_1.png` — `git secrets --list` (active AWS patterns)
- `phaseE_gitsecrets_2.png` — blocked commit on a fake AWS key
- `phaseE_gitsecrets_3.png` — `git secrets --scan-history` catching a bypassed commit

 Completion Checklist
- [x] Completed both PwnedLabs exercises
- [x] Captured screenshots of successful completion pages
- [x] Documented findings and remediation lessons (rotate leaked keys, use pre-commit secret scanning, never bake secrets into image layers or source code)
