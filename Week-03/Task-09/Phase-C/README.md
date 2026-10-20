Phase C: Access Control Testing

Objective

Test S3 access permissions, examine ACL limitations, and verify authenticated and anonymous access before and after applying a bucket policy.

Environment

- **Bucket:** `student-s3-lab-teetech-20261020`
- **IAM user and CLI profile:** `s3tester`
- **S3 object:** `confidential.txt`
- **Local source file:** `PhaseC_confidential.txt`
- **Object Ownership:** Bucket owner enforced; ACLs disabled
- **Block All Public Access:** Enabled

The sample file contains simulated lab data only.
1. Baseline Access Tests

 Upload the Sample File

```powershell
aws s3 cp PhaseC_confidential.txt s3://student-s3-lab-teetech-20261020/confidential.txt --profile s3tester
```

Result: Upload succeeded.

List the Bucket

```powershell
aws s3 ls s3://student-s3-lab-teetech-20261020 --profile s3tester
```

Result: The listing displayed `confidential.txt`.

 Download the Object

```powershell
aws s3 cp s3://student-s3-lab-teetech-20261020/confidential.txt downloaded-confidential.txt --profile s3tester
```

Result: Download succeeded.

Test Anonymous Reading

```powershell
aws s3api get-object --bucket student-s3-lab-teetech-20261020 --key confidential.txt anonymous-test.txt --no-sign-request
```

Result: AccessDenied.

The unsigned request confirmed that anonymous reading was denied before the bucket-policy change.

2. ACL Review and Task Limitation

I confirmed that the bucket used Bucket owner enforced Object Ownership, with ACLs disabled.

The task requested an object ACL granting read access to another IAM user within the same account. S3 ACLs cannot directly name individual IAM users as grantees. Account-level ACL grants use canonical account IDs.

I therefore retained ACLs disabled and documented the limitation. No ACL modification or second-user access test was performed. User-specific access would require an IAM policy or bucket policy.

 3. Explicit Anonymous-Access Denial

I applied the following bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAnonymousAccess",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::student-s3-lab-teetech-20261020",
        "arn:aws:s3:::student-s3-lab-teetech-20261020/*"
      ],
      "Condition": {
        "Null": {
          "aws:PrincipalArn": "true"
        }
      }
    }
  ]
}
```

The condition targets requests where `aws:PrincipalArn` is absent, including anonymous requests. It does not grant any permissions.

Block All Public Access remained enabled.

4. Tests After the Policy Change

I repeated authenticated bucket listing and anonymous object reading.

| Test | Before policy | After policy |
|---|---|---|
| Upload using s3tester | Allowed | Not repeated |
| Bucket listing using s3tester | Allowed | Allowed |
| Object download using s3tester | Allowed | Not repeated |
| Anonymous object reading | Denied | Denied |

 Key Observations

Authorized private access worked while anonymous access remained blocked.

The bucket was already private before the policy change. Adding the explicit denial did not remediate a previously public object.

Because multiple controls were active, the anonymous AccessDenied result alone does not prove which control caused the denial.

The authenticated listing test confirmed that the new policy preserved the tested user's listing access. It did not verify every authenticated identity or S3 action.

ACLs cannot directly grant access to an individual IAM user. Policy-based permissions are appropriate for that requirement.

## Evidence

- [Sample source file](PhaseC_confidential.txt)
- [Baseline access tests](PhaseC_baseline_access.png)
- [ACLs disabled](PhaseC_acls_disabled.png)
- [Tests after policy change](PhaseC_after_policy_tests.png)
- [CLI results log](phaseC_results.log)
 Completion Status

| Requirement | Status |
|---|---|
| Upload sample object | Completed |
| Modify object ACL for an individual IAM user | Not performed; technical limitation documented |
| Apply explicit anonymous-access denial policy | Completed |
| Test authenticated and anonymous access | Completed |
| Record before-and-after results | Recorded in CLI results log |
| Test access for another IAM user | Not performed |

References

- [S3 ACL Overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/acl-overview.html)
- [S3 Object Ownership](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html)
- [S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
