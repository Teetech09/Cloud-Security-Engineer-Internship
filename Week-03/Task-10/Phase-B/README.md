 Phase B: Homelab Environment Setup

Objective

Create a private S3 testing environment and a customer-managed KMS key for subsequent encryption testing.

Environment Configuration

| Component | Configuration |
|---|---|
| Bucket | secure-s3-lab-teetech-20261021 |
| Region | us-east-1 — US East (N. Virginia) |
| Bucket type | General purpose |
| Block All Public Access | Enabled |
| Object Ownership | Bucket owner enforced; ACLs disabled |
| Versioning | Enabled |
| Default bucket encryption | SSE-S3 |
| KMS key alias | alias/S3LabKey |
| KMS key type | Symmetric, single-Region |
| Key usage | Encrypt and decrypt |
| Key material origin | AWS KMS |
| Key status | Enabled |

 Actions Taken

I used the AWS Management Console to create the general purpose bucket `secure-s3-lab-teetech-20261021` in `us-east-1`.

I retained Bucket owner enforced Object Ownership with ACLs disabled and kept all four Block Public Access settings enabled.

I enabled versioning and retained SSE-S3 as the bucket's default encryption method.

I then opened KMS in the same Region and created a symmetric, single-Region customer-managed key for encryption and decryption. I assigned the alias `S3LabKey` and selected the appropriate IAM identities for key administration and usage. No external accounts were added.

No CLI commands, infrastructure templates or custom JSON policies were used. No new IAM identities or vulnerable environments were deployed.

Verification

| Check | Result |
|---|---|
| Bucket created | Confirmed in the S3 console |
| Block All Public Access | Enabled |
| ACLs | Disabled |
| Versioning | Enabled |
| Default encryption | SSE-S3 |
| KMS key | Created and enabled |
| Bucket and KMS key Region | Both us-east-1 |
| SSE-KMS applied to bucket | Not yet configured |
| KMS upload/download permissions | Not yet tested |

Key Observations

The setup completed without reported errors. Public access protection and disabled ACLs established a private baseline, while versioning was enabled to preserve object versions.

Creating the KMS key did not automatically apply SSE-KMS to the bucket. The bucket remained on SSE-S3, allowing later encryption and permission tests to be compared against this baseline.

Evidence

 Figure 1: Public Access Protection

![Block All Public Access enabled](PhaseB_Config.png)

Bucket permissions showing all public access blocks enabled.

 Figure 2: Versioning

![Bucket versioning enabled](PhaseB_Versioning.png)

Bucket properties confirming versioning is enabled.

Figure 3: KMS Key

![S3LabKey enabled](PhaseB_KMS_Key.png)

Customer-managed KMS key showing the alias and Enabled status.

Completion Checklist

- [x] S3 bucket created.
- [x] Public access blocked.
- [x] Versioning enabled.
- [x] Customer-managed KMS key created.
