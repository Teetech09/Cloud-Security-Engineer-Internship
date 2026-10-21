Phase A: Theory and Introduction

Objective

Understand the risks of misconfigured S3 buckets and how encryption and access controls help mitigate them.

Actions Taken

- Reviewed AWS documentation on S3 security best practices and SSE-KMS.
- Examined public-read access, public-write access and exposed credentials.
- Compared SSE-S3 with SSE-KMS.
- Studied the relationships between IAM roles, bucket policies and object ACLs.
- Reviewed three documented open-bucket exposure cases.
- Used ChatGPT to explain concepts and draft the summary content.

No AWS resources were modified during this phase.

Common Misconfigurations

| Misconfiguration | Potential Impact | Mitigation |
|---|---|---|
| Unintended public read | Disclosure of sensitive objects | Enable Block Public Access and remove public grants |
| Unintended public write | Unauthorized uploads or overwrites, depending on permissions | Restrict write actions to approved identities |
| Overly broad IAM permissions | Compromised identities access unnecessary resources | Scope actions and resources using least privilege |
| Credentials in accessible files | Credential reuse against other resources | Remove exposed files, rotate keys and use temporary credentials |

 SSE-S3 vs SSE-KMS

| Feature | SSE-S3 | SSE-KMS |
|---|---|---|
| Key management | Amazon S3 manages keys | Uses AWS KMS keys |
| Access authorization | S3 permissions | S3 permissions and applicable KMS permissions |
| Customer-managed key policies | Not applicable | Supported with customer-managed KMS keys |
| Key-use auditing | No customer-visible KMS key-use events | KMS operations can be audited through CloudTrail |
| Additional encryption cost | None | KMS charges can apply |

New S3 uploads are encrypted at rest automatically. Encryption does not replace appropriate access controls.

SSE-KMS object downloads require applicable kms:Decrypt permission. Customer-managed keys provide additional governance but also require careful permission and lifecycle management.

IAM Roles, Bucket Policies and ACLs

- **IAM roles:** Assumable identities that use temporary credentials; attached policies define their permissions.
- **Bucket policies:** Resource-based JSON policies controlling access to a bucket and its objects.
- **Object ACLs:** Legacy grants associated with individual objects when ACLs are enabled.

Applicable controls are evaluated together. An explicit deny overrides an allow.

AI-Assisted Summary: Bucket Policy vs Object ACL

A bucket policy can specify principals, actions, resources, conditions and explicit denies. Although attached to a bucket, it can target individual objects.

An object ACL provides a limited set of grants for one object. It does not support policy-style conditions or explicit denies and cannot directly name an individual IAM user as a grantee.

Bucket policies provide finer control than ACLs. Object-specific access does not require enabling ACLs, and AWS recommends keeping ACLs disabled for most use cases.

AI tool used: ChatGPT.

Real-World Exposure Cases

| Case | Documented Exposure | Security Lesson |
|---|---|---|
| Verizon partner, 2017 | Customer information in a publicly accessible S3 repository | Review third-party storage permissions |
| Accenture, 2017 | Sensitive information, including credentials, in publicly accessible buckets | Restrict storage access and prevent credential publication |
| Booz Allen Hamilton/NGA, 2017 | Sensitive files and access credentials in an unsecured S3 repository | Storage exposure can create access risks beyond the bucket |

These are documented exposure cases. Public accessibility alone does not establish malicious exploitation.

Key Observations

Encryption, access control and monitoring provide complementary protection.

SSE-KMS adds key-level authorization and governance, while SSE-S3 provides straightforward encryption at rest.

A denied request does not prove that every access path is secure. Permissions must be evaluated across identities, objects and request conditions.

Deliverable

- [One-page Phase A summary](PhaseA_Summary.pdf)

 References

- [AWS S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)
- [AWS SSE-KMS Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)
- [S3 ACL Overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html)
- [Verizon Partner Exposure — UpGuard](https://www.upguard.com/breaches/verizon-cloud-leak)
- [Accenture Exposure — UpGuard](https://www.upguard.com/breaches/cloud-leak-accenture)
- [Booz Allen Hamilton/NGA Exposure — UpGuard](https://www.upguard.com/breaches/spy-games-booz-allen-hamilton-pentagon)
