Phase E: Hosted Labs Integration

 Objective
Reinforce the Phase B to D S3 work through two external hosted labs: one on basic S3 enumeration and one on identifying the AWS account that owns a public S3 bucket.

 Evidence
| File | Description |
|------|-------------|
| [`phaseE_cybrlab_completion.png`](./phaseE_pwnedlab_completion.png) | Completion screenshot for the Cybr and PwnedLabs labs |

Labs Completed

| Platform | Lab | Type | Focus |
|----------|-----|------|-------|
| Cybr | [Introduction to S3 Enumeration](https://cybr.com/hands-on-labs/lab/introduction-to-s3-enumeration/) | Capture the flag | Listing and downloading S3 data with the AWS CLI |
| PwnedLabs | [Identify the AWS Account ID from a Public S3 Bucket](https://pwnedlabs.io/labs/identify-the-aws-account-id-from-a-public-s3-bucket) | Beginner-friendly, AWS | Account ID disclosure through a public bucket |

 Actions Taken

Cybr: Introduction to S3 Enumeration
Objective: learn how to enumerate S3 with the AWS CLI, which has quirks compared with other AWS services, and capture the flag held in `object.txt`.

Configured the AWS CLI with the access keys supplied by the lab (`aws configure`) and confirmed the identity with `aws sts get-caller-identity`. The identity was the IAM user **Derek**.
Read the IAM user policy (`aws iam list-user-policies`, `aws iam get-user-policy`) to see which S3 actions were allowed:
  - `s3:ListAllMyBuckets` and `s3:GetBucketLocation` to list buckets and see their region.
  - `s3:ListBucket` and `s3:GetBucketPolicy` on the two lab buckets to list objects and read the bucket policy.
  - `s3:GetObject` on `bucket/*` to read object contents.
- Listed buckets and objects with `aws s3 ls` and `aws s3 ls s3://<bucket>`.
- Downloaded an object with `aws s3 cp` and read it with `cat` to obtain the flag. [Add: which bucket the object was in.]
- Examined why a bucket policy can block access that an IAM policy allows. Access is evaluated across identity policies and resource policies, and an explicit `Deny` in either overrides any `Allow`. [Add: whether one bucket denied you and what you saw when you ran `aws s3api get-bucket-policy`.]

 PwnedLabs: Identify the AWS Account ID from a Public S3 Bucket
Objective: Starting from the IP address of a logistics company's website, identify its AWS account ID through a public S3 bucket, then understand the technique by reviewing the source code of the tool rather than only running it.

- Started the engagement and used the provided IP address to find the public S3 bucket. [Add: how you identified the bucket.]
- Identified the account ID using the technique taught in the lab. [Add: the tool or command you used and a redacted result.]
- Reviewed the tool's code to understand how it works. The technique relies on how AWS evaluates policy conditions that compare the resource owner with a value that supports wildcards. Each request succeeds or fails depending on whether the condition matches, which turns policy evaluation into a yes or no answer. Testing one digit position at a time recovers the account ID without credentials for the target account.

Key Observations

- **Object access and bucket access are separate permissions.** `s3:ListBucket` shows object names, while `s3:GetObject` on `bucket/*` is what lets someone read the data. This is why least privilege for S3 must be applied at both levels.
- **Policies are layered.** An IAM allow does not guarantee access, because bucket policies, permissions boundaries and SCPs can deny it. Explicit denies are a useful extra control on top of Block Public Access.
- **An account ID is not a secret, but it enables reconnaissance.** With the ID, an attacker can test whether specific IAM roles and users exist without authenticating, turning guessable naming conventions into a confirmed target list. The lab maps this to MITRE ATT&CK T1580 (Cloud Infrastructure Discovery) and T1087 (Account Discovery).
- **Discovery is hard to detect from the target side.** The lab recommends assuming the account ID is discoverable if anything is public, and designing so that knowing it gives no advantage. Detection effort is better spent on failed role assumption and sign-in attempts against named principals.

Link to Phase D Risk Matrix
- Account ID disclosure is reconnaissance that precedes T2 (credential theft) and T3 (over-permissive IAM) in the Phase D matrix. Predictable role names make the later attacks easier.
- Bucket exposure (T1) is the starting condition for both labs. Block Public Access and non-guessable identity names would reduce what an attacker learns.
- Least privilege (T3) is shown in practice by Derek's policy, where object read access is limited to two named buckets.

Mitigations Reinforced
1. Enable S3 Block Public Access at the account level wherever public access is not a deliberate requirement.
2. Scope `s3:GetObject` and `s3:ListBucket` to specific bucket ARNs.
3. Use non-guessable components in IAM role and user names.
4. Serve public content through a distribution layer, such as CloudFront or presigned URLs, instead of exposing buckets directly.
5. Monitor for failed role assumption attempts that reference principals in your accounts.

 Completion Checklist
- [x] Cybr: Introduction to S3 Enumeration completed
- [x] PwnedLabs: Identify the AWS Account ID from a Public S3 Bucket completed
- [x] Completion evidence captured (`phaseE_lab_completion.png`)

> Flag values and access keys are intentionally not included in this repository.
