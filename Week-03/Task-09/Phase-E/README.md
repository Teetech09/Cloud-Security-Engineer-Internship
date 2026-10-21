Phase E: Hosted Labs Integration

Objective

Apply S3 enumeration and access-analysis techniques in authorized PwnedLabs environments, and investigate how exposed credentials can lead to further cloud access.

Labs Completed

1. AWS S3 Enumeration Basics
2. S3 Bucket Brute Force to Breach

Both labs were completed, and completion screenshots were saved.

Lab 1: AWS S3 Enumeration Basics

Activities and Results

- Enumerated the lab bucket anonymously using the AWS CLI.
- Compared root, prefix-specific and recursive listing requests.
- Downloaded a publicly accessible migration archive and inspected its PowerShell script.
- Identified hardcoded lab credentials and configured separate named CLI profiles.
- Verified each credential pair's identity using AWS STS.
- Retrieved a migration XML file and identified another lab identity.
- Confirmed access to the lab's transaction export using the verified it-admin credentials.
- Retrieved and examined the bucket policy.
- Verified that anonymous browser listing worked when the URL included the required prefix and delimiter parameters.
- Captured and submitted the lab flag.

Key Observations

A failed browser request did not prove that the bucket was private. Anonymous listing depended on the request matching the bucket policy's prefix and delimiter conditions.

Listing and object reading were separate permissions. Knowing an object key could allow a read attempt even when a particular listing request was denied.

Credentials exposed in downloadable files enabled authenticated access beyond the initial anonymous access.

A CLI profile name did not establish the identity behind it. STS verification was necessary to confirm which IAM user the stored credentials represented.

Troubleshooting

| Issue | Resolution |
|---|---|
| PowerShell interpreted curl as Invoke-WebRequest | Used curl.exe |
| Linux unzip command differed from Windows tooling | Used Expand-Archive |
| Commands used unintended default credentials | Specified the appropriate named profile |
| Incorrect credentials resolved to pam-test | Located the matching it-admin credential pair and verified it using STS |
| Fresh credential configuration was needed | Created a clean named profile |
| Dotted bucket hostname caused a certificate mismatch | Used an HTTPS regional path-style URL |
| Intermittent endpoint connection failures | Retried requests and used an explicit Region where appropriate |

Lab 2: S3 Bucket Brute Force to Breach

 Activities and Results

- Identified eu-west-2 as the initial bucket's Region from the response headers.
- Installed Go and ffuf on Windows.
- Downloaded the lab wordlist and created regions.txt.
- Ran the lab-prescribed bucket-name enumeration.
- Manually investigated the intended lab buckets using unsigned AWS CLI requests.
- Listed and viewed index.html from hlogistics-web.
- Downloaded and inspected SystemTrackingPackagesTest.py from hlogistics-beta.
- Configured the discovered ecollins lab credentials and verified the identity.
- Enumerated the SSM_Parameter inline IAM policy.
- Retrieved the specifically permitted lharris parameter from Systems Manager Parameter Store.
- Configured the new lab credentials and verified the lharris identity.
- Completed the remaining lab steps and captured the flag.

Enumeration Results

| Candidate Bucket | Region | HTTP Status |
|---|---|---|
| hlogistics-images | eu-west-2 | 200 |
| hlogistics-web | eu-west-2 | 200 |
| hlogistics-storage | us-east-1 | 403 |
| hlogistics-beta | eu-west-2 | 200 |
| hlogistics-staticfiles | eu-west-2 | 200 |

The scan completed 54,671 requests and reported 20,538 errors. Coverage was therefore incomplete. The output alone did not establish the cause of every error.

HTTP response codes were treated as investigation leads, rather than proof of company ownership or object-read permissions.

IAM Policy Analysis

The ecollins inline policy included:

- ssm:GetParameter
- ssm:DescribeParameters

Its resource was restricted to the lharris parameter in eu-west-2.

The GetParameter permission allowed retrieval of that parameter. DescribeParameters requires Resource "*", so the scoped statement alone did not grant general parameter discovery.

Key Observations

The lab demonstrated a chain from publicly readable source code to exposed credentials, then to a readable Parameter Store value containing another identity's credentials.

Parameter Store security depended on the permissions of identities that could retrieve its values. Storage in a managed secrets-related service did not prevent disclosure to an authorized but compromised identity.

IAM usernames did not establish privilege levels. Identity and permissions needed to be verified separately.

Troubleshooting

| Issue | Resolution |
|---|---|
| S3 hostname DNS failures | Used a working regional path-style endpoint |
| Go was unavailable in PowerShell | Installed Go and reopened PowerShell |
| ffuf was not on PATH | Invoked its executable using the full path |
| Beta bucket endpoint connection failed | Specified the regional endpoint URL |
| aws-enumerator executable was unavailable during troubleshooting | Recorded the tooling issue; no successful tool output is claimed here |

Security Lessons

- Keep non-public buckets protected with Block Public Access.
- Review bucket policy conditions rather than relying on one failed request.
- Avoid embedding credentials in scripts, archives or source code.
- Prefer IAM roles and temporary credentials where appropriate.
- Restrict Parameter Store access to necessary parameters and identities.
- Rotate exposed credentials and investigate their use.
- Monitor sensitive S3 object access and parameter retrieval.
- Verify identity using STS when switching credential profiles.

Evidence

Lab 1 Completion

![AWS S3 Enumeration Basics completion](PhaseE_pwnedlab1_completion.png)

 Lab 2 Completion

![S3 Bucket Brute Force to Breach completion](PhaseE_pwnedlab2_completion.png)

Credentials, parameter secret values, transaction contents and flags are excluded from this public documentation.

Scope

All activities were performed within the authorized PwnedLabs training scope.

Lab References

- [AWS S3 Enumeration Basics](https://pwnedlabs.io/labs/aws-s3-enumeration-basics)
- [S3 Bucket Brute Force to Breach](https://pwnedlabs.io/labs/s3-bucket-brute-force-to-breach)
