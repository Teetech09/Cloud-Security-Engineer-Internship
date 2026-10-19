Phase E – Hosted Labs: IAM Credential Review and Credential Abuse

Objective

Understand how to review AWS credential security using IAM credential reports and explore how compromised or reused credentials can expose AWS accounts.

Lab 1: Cybr – Generate IAM Credentials Reports

Lab link: https://cybr.com/hands-on-labs/lab/generate-iam-credentials-reports/

Status: Completed

 Activities Completed

- Generated an IAM credential report through the AWS Console.
- Reviewed which users had console passwords, access keys, and MFA.
- Generated and retrieved the same report using the AWS CLI.
- Decoded the Base64 report content into readable CSV text.
- Reviewed the report for potential credential security problems.

Commands Used

```bash
aws iam generate-credential-report
aws iam get-credential-report
aws iam get-credential-report --output text --query Content | base64 -d
```

 Report Interpretation

The report provides information about password availability and usage, MFA status, access-key activity, and credential rotation dates.

In the provided lab example:

- `ReportFormat` showed `text/csv`.
- The root account’s `password_next_rotation` field showed `not_supported`.

The `not_supported` value does not, by itself, indicate a security weakness.

 Security Lessons

Credential reports help identify accounts that require investigation, such as password-enabled users without MFA, unused active access keys, or root access keys. These conditions must be verified against the actual report before being documented as findings.

Lab 2: PwnedLabs – Execute and Identify Credential Abuse in AWS
 Lab link: https://pwnedlabs.io/labs/execute-and-identify-credential-abuse-in-aws

Status: Partially completed

 Activities and Progress

- Worked on setting up the local tools required for the credential-recovery stage.
- Confirmed that John the Ripper was installed in Ubuntu.
- Reviewed the walkthrough’s use of John the Ripper to recover passwords from SHA-256 hashes.
- Reviewed how recovered credentials could be tested for reuse against AWS Console accounts within the lab.
- Attempted to locate the required `user_hash` file.

 Blocker

The expected input file was not found at `/home/teetech/user_hash`. A filename search did not locate the required lab file.

An attempt to install GoAWSConsoleSpray also encountered a missing Go installation. The tool was not successfully run during the recorded session.

Verification Status

- John the Ripper installation: confirmed.
- Successful recovery of the lab passwords: not verified.
- AWS Console credential testing: not verified.
- Successful AWS Console sign-in: not verified.

The successful username and password shown in the walkthrough are example results and are not presented as my own findings.

Security Lessons

Offline password-hash cracking and online credential testing are separate activities. Weak passwords and password reuse can expose additional accounts when credentials are compromised. MFA and unique passwords help reduce this risk.

Combined Lessons Learned

- IAM credential reports support preventive reviews of AWS account security.
- Credential-abuse exercises demonstrate the possible consequences of weak or reused passwords.
- Tools must have the correct input files and dependencies before testing can proceed.
- Documentation must distinguish observed results from walkthrough examples and unverified outcomes.

 Evidence

Attach available evidence for:

- Cybr lab completion.
- IAM credential report generation and retrieval.
- Decoded credential report.
- John the Ripper installation.
- Missing hash-file error and other setup blockers.

Redact credentials and other sensitive information before uploading.

 Overall Outcome

Completed the Cybr IAM credential-report lab. The PwnedLabs credential-abuse lab remains partially completed because of input-file and tooling blockers. Password recovery and AWS Console access were not verified.
