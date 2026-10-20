Phase B: Environment Setup

Objective

Create a private S3 testing environment and configure an IAM user with read/write permissions restricted to the test bucket.

Environment Configuration

| Component | Configuration |
|---|---|
| S3 bucket | student-s3-lab-teetech-20261020 |
| Block All Public Access | Enabled |
| Object Ownership | Bucket owner enforced; ACLs disabled |
| Versioning | Enabled |
| Default encryption | SSE-S3 |
| IAM user | s3tester |
| Console access | Disabled |
| Access method | AWS CLI using an access key |
| CLI profile | s3tester |
| Custom inline policy | S3TesterLabReadWrite |

Actions Taken

I created the private S3 bucket, kept all public access blocks enabled and ACLs disabled, and enabled versioning with SSE-S3 encryption.

I created the IAM user `s3tester` without console access and attached a custom inline policy granting list, read and upload permissions only within the test bucket.

I configured the named CLI profile `s3tester`, verified its identity using AWS STS, and successfully listed the bucket.

### Interpretation of the Public Access Instruction

The task wording “disable public access block overrides” was ambiguous. To meet the stated objective of creating a private environment, I kept all four Block Public Access settings enabled.

IAM Policy

The policy separates bucket-level permissions from object-level permissions.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListTestBucket",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::student-s3-lab-teetech-20261020"
    },
    {
      "Sid": "ReadWriteTestObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::student-s3-lab-teetech-20261020/*"
    }
  ]
}

This policy grants no object deletion, bucket configuration or access-management permissions. It does not grant access to other buckets.
CLI Verification
Configure the Named Profile
aws configure --profile s3tester

Credentials were entered locally and are excluded from this repository.
Verify the Identity
aws sts get-caller-identity --profile s3tester

Result: The returned ARN identified user/s3tester.
Test Bucket Listing
aws s3 ls s3://student-s3-lab-teetech-20261020 --profile s3tester

Result: The command completed without an access error. The empty bucket returned no object listings.
Validation Status
Check	Status
CLI identity verification	Passed
Test bucket listing	Passed
Object upload	Not yet tested
Object read/download	Not yet tested
Access denial outside the test bucket	Not yet tested


Key Observations
The bucket ARN applies to s3:ListBucket, while the object ARN ending in /* applies to object reading and uploading.

Block Public Access does not prevent authorized private access through an IAM policy.

Versioning preserves object versions and supports recovery from overwrites. It does not replace CloudTrail API activity logging.

The successful listing validates listing permission only. Uploading, reading and access restrictions require separate tests.
Evidence
- [Bucket creation settings](PhaseB_bucket_creation.png)
- [Versioning and encryption settings](PhaseB_versioning_encryption.png)
- [IAM policy JSON](phaseB_policy.json)
- [CLI verification](PhaseB_cli_verification.png)
References
- Creating an S3 Bucket
- S3 Block Public Access
