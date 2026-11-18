Task 29 – Phase E: Defensive Engineering (Hardening & Detection Validation)

Objective

Turn the flaws.cloud findings from Levels 1–6 into defensive controls. For each misconfiguration, identify the root cause, define preventive and detective controls, and validate the hardening in my own sandbox account.


Actions Taken
Built the hardening matrix (PhaseE_HardeningMatrix.xlsx, sheet Hardening Matrix). It has six rows, one per finding, each with root cause, preventive control, detective control, AWS service, priority and validation method.

Wrote hardened policy examples (hardened_policy_examples.json) covering S3 access restriction, credential and secret handling with Secrets Manager, EBS snapshot sharing, and a bonus on IMDSv2 and least-privilege roles.

Created a detection plan (sheet Detection Plan) with 5 use cases. It separates CloudTrail management events from S3 data events, because object-level access (such as Level 1 listing) only appears if data events are enabled.

Completed the sandbox review (sheet Sandbox Review) and executed the hardening commands against my sandbox:

Enabled account-level S3 Block Public Access:
aws s3control put-public-access-block --account-id <SANDBOX_ID> --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true --profile flaws-sandbox

Enabled EBS snapshot block public access in us-west-2:
aws ec2 enable-snapshot-block-public-access --state block-all-sharing --region us-west-2 --profile flaws-sandbox

Removed the challenge credentials from the local AWS CLI config for flaws-l2, flaws-l3-leaked, flaws-l5-ssrf and flaws-l6-audit, including the Level 5 session token.

Verified each change with read-back commands:
aws s3control get-public-access-block ... returned all four settings as true.
aws ec2 get-snapshot-block-public-access-state ... returned block-all-sharing, ManagedBy: account.



Key Observations
Level	Root cause	Preventive control	Detective control
1	Public bucket listing	S3 Block Public Access, private bucket policy	CloudTrail S3 data events, Config rule s3-bucket-public-read-prohibited
2	Bucket open to any authenticated AWS user	Restrict principals to specific accounts or roles	CloudTrail data events, IAM Access Analyzer
3	Access key committed to a .git history	Secrets Manager, pre-commit secret scanning, key rotation	Detect use of the key from unexpected accounts or IPs
4	Publicly shared EBS snapshot	Snapshot block public access, encryption with a CMK	CloudTrail ModifySnapshotAttribute alerts
5	SSRF reaching the instance metadata service (IMDSv1)	Require IMDSv2, restrict the proxy, least-privilege role	GuardDuty, CloudTrail anomalous role use
6	Over-broad audit permissions exposing the API surface	Least privilege, scoped read-only policies	CloudTrail IAM and Lambda enumeration calls
Block Public Access is a single account-level control that closes the Level 1 and 2 exposure classes.
EBS snapshot block public access is regional, so it must be enabled in every region in use. I enabled it in us-west-2 only.
Clearing local credentials does not invalidate them. The IAM keys must also be deactivated or deleted on the AWS side.

Evidence Produced
    • Final_Report.pdf 
    


Evidence Produced
PhaseE_HardeningMatrix.xlsx (Hardening Matrix, Detection Plan, Sandbox Review sheets)
hardened_policy_examples.json
[your screenshot filename].png (sandbox hardening verification)
Command output for get-public-access-block and get-snapshot-block-public-access-state
