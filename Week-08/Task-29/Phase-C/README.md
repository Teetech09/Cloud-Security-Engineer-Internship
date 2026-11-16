Phase C: Core Execution — flaws.cloud Levels 1-3

Actions Taken

I worked through Levels 1 to 3 of flaws.cloud independently, without reading any solution walkthroughs, and recorded the full session in `PhaseC_Level1-3_CommandLog.txt`.

For Level 1, I ran DNS reconnaissance against flaws.cloud with `dig`, `nslookup` and `host`, which returned a set of IP addresses consistent with Amazon S3 static website hosting rather than a conventional web server. I then tested whether the bucket could be listed anonymously using `aws s3 ls --no-sign-request`, which succeeded and returned seven objects, including a file that was not linked from the site's `index.html`. I confirmed via `robots.txt` that this file had only been excluded from search engine crawling, not from direct access, and retrieving it anonymously revealed the Level 2 URL.

For Level 2, I repeated the anonymous listing attempt against the Level 2 bucket, which returned Access Denied, and then ran the identical command using my sandbox AWS CLI profile (an authenticated, non-root IAM principal), which succeeded. This direct comparison showed that the bucket was not public, but was open to any authenticated AWS identity. Reading the revealed secret file gave me the Level 3 URL.

For Level 3, I synced only the Level 3 bucket into its own working folder and found that a `.git` directory had been published alongside the site content. Reviewing `git log --all --oneline` showed two commits, and `git log --all -p` revealed that the first commit added a file named `access_keys.txt` containing an AWS access key and secret key, and the second commit removed that file. Because Git retains all prior history, the credentials remained fully recoverable from the earlier commit despite the deletion, a point the author's own commit message ("Oops, accidentally added something I shouldn't have") makes explicit. I captured this history to a text file and redacted the key and secret before saving it as evidence, then deleted the unredacted copy. I validated the leaked credential by configuring it under a new, separate named profile rather than reusing any of my own sandbox profiles, and confirmed with `aws sts get-caller-identity` that it belonged to a distinct AWS account under an IAM user named `backup`. I then ran a single `aws s3 ls` under that identity, which is the minimum needed to discover the Level 4 bucket name, and stopped there rather than exploring any further resources that credential could reach.

Key Observations

Across all three levels, the common thread was that AWS access controls operate at a finer grain than people often assume, and each level demonstrates a different way that gap gets exploited.

Level 1 showed that granting public read access to serve a static website (`s3:GetObject`) is a separate permission from allowing the bucket to be listed (`s3:ListBucket`). A site can be intentionally public to visitors while still not intending for its full object listing to be exposed, since listing reveals files that were never linked from any page and were never meant to be publicly discoverable. A `robots.txt` disallow entry is a request to well-behaved crawlers, not an access control, so it offered no real protection once the bucket could be listed directly.

Level 2 showed that AWS's `AuthenticatedUsers` grantee group is frequently misread as meaning "users in my organization" or "people I trust," when it actually means any principal with valid AWS credentials in any AWS account whatsoever. Granting access to this group is functionally close to public access, since anyone can create a free AWS account and sign a request.

Level 3 showed that deleting a file in a later Git commit does not remove it from the repository's history; every prior version remains retrievable through the commit log unless history itself is rewritten. Publishing a `.git` directory as part of a static site's bucket content is, in effect, publishing the project's entire change history alongside the files, which is exactly how the leaked credential was recovered here. It also reinforced why long-lived IAM access keys are risky when exposed: the key kept working in validation, with no built-in expiry, until someone with permission revokes it.

Handling the Level 3 credential carefully mattered as much as finding it. Validating it in a dedicated, separate profile kept it clearly isolated from my own account's credentials, and limiting enumeration to a single `s3 ls` call was enough to find the Level 4 bucket without unnecessarily exploring everything that identity could reach.

Findings

| Level | AWS Service / API | Misconfiguration | Business Impact | MITRE ATT&CK for Cloud | Preventive Controls |
|---|---|---|---|---|---|
| 1 | S3 / `ListObjectsV2` | Bucket allowed anonymous `s3:ListBucket`, not limited to `s3:GetObject` on the website's own files | Any visitor can enumerate every object in the bucket, including unlinked and unindexed files, exposing internal structure | Reconnaissance: Search Victim-Owned Websites; Discovery: Cloud Storage Object Discovery | (1) Enable S3 Block Public Access at the bucket or account level; (2) scope any public grant to `s3:GetObject` on specific object keys, never `s3:ListBucket` |
| 2 | S3 / `ListObjectsV2` | Bucket policy granted access to the `AuthenticatedUsers` group rather than named principals | Any AWS account holder worldwide, not just trusted parties, can access the bucket's contents | Initial Access: Valid Accounts – Cloud Accounts | (1) Scope bucket policies to specific IAM principals or account IDs using the `Principal` element; (2) use conditions such as `aws:PrincipalAccount` instead of relying on the broad `AuthenticatedUsers` grantee |
| 3 | S3 (object storage of `.git`) / IAM (leaked access key) | A `.git` directory containing commit history with a hardcoded access key and secret was published in the public bucket | Leaked long-lived IAM credentials grant whatever that key's attached policy allows, until the key is manually deactivated or rotated; an attacker gains that access the moment they find it | Credential Access: Unsecured Credentials (credentials in files); Initial Access: Valid Accounts – Cloud Accounts | (1) Exclude `.git` and other VCS directories from anything deployed to a public bucket, and scan repositories with tools such as Gitleaks or TruffleHog before publishing; (2) never commit credentials to source control — use environment variables, AWS Secrets Manager, or short-lived STS tokens, and rotate/deactivate immediately if a key is ever exposed |

Evidence Produced

- PhaseC_Level1-3_CommandLog.txt: full terminal session log for all three levels, checked to confirm it contains no access keys or secrets.
- PhaseC_Level1_AnonList.png: anonymous `aws s3 ls` listing of the flaws.cloud bucket.
- PhaseC_Level1_SecretFile.png: contents of the unlinked secret file revealing the Level 2 URL.
- PhaseC_Level2_AuthCompare.png: side-by-side comparison of the denied anonymous listing and the successful authenticated listing of the Level 2 bucket.
- PhaseC_Level2_SecretFile.png: contents of the Level 2 secret file revealing the Level 3 URL.
- PhaseC_Level3_GitFolderExposed.png: directory listing showing the published `.git` folder.
- PhaseC_Level3_GitHistory_redacted.txt: redacted Git history showing the credential added and later removed, with the access key and secret masked.
- PhaseC_Level3_BucketEnum.png: minimal bucket enumeration under the validated leaked credential, showing the Level 4 bucket name.
