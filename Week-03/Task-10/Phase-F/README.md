Phase F: Documentation and Reporting

Objective

Consolidate task evidence into a professional report demonstrating secure S3 configuration, encryption, access control and auditability.

Actions Taken

I consolidated the configurations, commands, screenshots and observations from Phases A–E into report content.

The documentation covers the private S3 environment, customer-managed KMS key, SSE-KMS upload and download, scoped bucket-policy grants, anonymous access denial and verified server access logs.

I included a threat model and recorded the ACL limitation, premium-lab restriction and remaining validation gaps.

Configuration and Validation Summary

| Component / Test | Result |
|---|---|
| Source bucket | secure-s3-lab-teetech-20261021 |
| Audit bucket | secure-s3-audit-teetech-20261021 |
| Region | us-east-1 |
| Public access protection | Enabled on both buckets |
| Object Ownership | ACLs disabled |
| Source bucket versioning | Enabled |
| Source bucket default encryption | SSE-S3 |
| KMS key | alias/S3LabKey |
| Test-object SSE-KMS upload | Successful |
| SSE-KMS object download | Successful |
| Anonymous object access | AccessDenied |
| Server access logging | Enabled; delivery verified |
| Recorded upload | REST.PUT.OBJECT — 200 |
| Recorded download | REST.GET.OBJECT — 200 |

Threat Model

| Attack Vector | Potential Impact | Applied Controls | Remaining Improvement |
|---|---|---|---|
| Unintended public access | Disclosure of stored data | Block Public Access; ACLs disabled; anonymous denial tested | Monitor permission and public-access changes |
| Compromised ops-user credentials | Access within the identity's effective permissions | Scoped bucket-policy grants; identity verification | Review all effective permissions and reduce unnecessary privileges |
| KMS key misuse or unavailability | Unauthorized decryption or loss of legitimate access | Key-user configuration; encrypted upload and download tested | Review key administrators, users and lifecycle permissions |
| Uploads without the intended KMS key | Loss of required key-specific governance | SSE-KMS explicitly selected for test uploads | Enforce the approved key if required |
| Delayed detection or loss of audit evidence | Reduced investigation capability | Server access logs delivered and inspected | Define alerts, retention and protection against log deletion |

These are assessed attack paths, not evidence of an actual compromise.

Key Observations

Encryption, access control and logging provided complementary protection.

The successful upload and download demonstrated functional SSE-KMS access for the tested identity. Delivered logs independently confirmed the read/write activity.

The bucket policy did not establish exclusive access or overall least privilege because permissions granted elsewhere were not fully assessed.

 Limitations and Task Deviations

- The private ACL command was rejected because Bucket owner enforced disables ACLs. This setting was retained.
- SSE-KMS was selected per object; the bucket default remained SSE-S3.
- The second Cybr lab required premium access and was not completed.
- Logging delivery was asynchronous.
- Automated alerting and additional log-retention protections were not configured during this task.

Recommended Improvements

- Review the complete effective permissions of ops-user.
- Enforce the intended KMS key if all future uploads must use it.
- Monitor bucket-policy, public-access and KMS configuration changes.
- Define audit-log retention and access restrictions.
- Use a dedicated role with temporary credentials for routine testing where practical.

 Report Deliverable

**Required filename:** SecureS3_Report.pdf

Upload the exported and reviewed PDF to this folder, then add its download link here.

 Evidence Handling

Credentials and secret values are excluded from public documentation. Account identifiers and personal IP addresses are redacted from published log evidence.

AI Assistance

ChatGPT assisted with explanations, troubleshooting, command adaptation, analysis and drafting. AWS configurations and lab commands were performed by me, with results checked against the recorded evidence.
