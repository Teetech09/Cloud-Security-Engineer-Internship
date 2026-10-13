Phase D — Least-Privilege Policy Redesign and Validation

Objective
Replace broad S3 read permissions with bucket-scoped access, validate permission decisions through simulation, and document a short threat model.

Actions Taken
I created an inline policy named `InternLabBucketReadOnly` on `intern_group`. I then removed the group's `AmazonS3ReadOnlyAccess` attachment so that its broad permissions would no longer apply through that group.

The replacement policy allows listing and reading only within `encrypted-identity-lab`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListLabBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::encrypted-identity-lab"
    },
    {
      "Sid": "ReadLabObjects",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::encrypted-identity-lab/*"
    }
  ]
}
```

CLI Simulation
In AWS CloudShell, I retrieved the lab user's ARN:

```bash
INTERN_USER_ARN=$(aws iam get-user \
  --user-name intern_user \
  --query 'User.Arn' \
  --output text \
  --no-cli-pager)
```

I simulated reading inside the permitted bucket:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn "$INTERN_USER_ARN" \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::encrypted-identity-lab/test-object.txt \
  --output json --no-cli-pager > phaseD-allowed-read.json
```

I simulated reading outside the permitted bucket:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn "$INTERN_USER_ARN" \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::phase-d-outside-scope-example/test-object.txt \
  --output json --no-cli-pager > phaseD-denied-outside-bucket.json
```

I simulated upload and deletion inside the lab bucket:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn "$INTERN_USER_ARN" \
  --action-names s3:PutObject s3:DeleteObject \
  --resource-arns arn:aws:s3:::encrypted-identity-lab/test-object.txt \
  --output json --no-cli-pager > phaseD-denied-write-delete.json
```

I simulated listing the lab bucket:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn "$INTERN_USER_ARN" \
  --action-names s3:ListBucket \
  --resource-arns arn:aws:s3:::encrypted-identity-lab \
  --output json --no-cli-pager > phaseD-allowed-list.json
```

Observed Results

| Action | Resource Tested | Result |
|---|---|---|
| `s3:ListBucket` | Lab bucket | `allowed` |
| `s3:GetObject` | Object inside lab bucket | `allowed` |
| `s3:GetObject` | Object outside lab bucket | `implicitDeny` |
| `s3:PutObject` | Object inside lab bucket | `implicitDeny` |
| `s3:DeleteObject` | Object inside lab bucket | `implicitDeny` |

The allowed results matched `group_intern_group_InternLabBucketReadOnly`, confirming that the evaluated permission came from the group's inline policy.

The denied results contained no matching allow statements. These were implicit denials, rather than explicit Deny statements.

All tests were simulations. No live objects were read, uploaded or deleted, and the example object and outside-bucket names did not need to exist.

Threat Model

| Asset | Threat | Potential Impact | Mitigation and Status |
|---|---|---|---|
| S3 objects | Compromised credentials used with broad read permissions | Sensitive data exposure — confidentiality | Implemented bucket-scoped list and read permissions. |
| S3 objects | Unnecessary upload or delete permissions | Data alteration or loss — integrity and availability | Simulation confirmed upload and deletion were implicitly denied. |
| EC2 instances and IAM roles | Excessive role permissions or misconfigured trust enable privilege escalation | Unauthorised control of cloud resources | Earlier review found EC2 service trust and DescribeInstances-only permissions on `intern_role`. No escalation path was demonstrated. |
| IAM identities | Credential theft | Unauthorised use of the identity's permissions | Recommend MFA for interactive access, temporary credentials where practical, and regular access reviews. MFA enforcement for `intern_user` was not tested. |

The threat model is saved as `threat_model.pdf`.

 Key Observations
- Adding a narrower Allow policy does not override an existing broad Allow policy; the broad attachment needed to be removed.
- Bucket-level and object-level actions require different resource ARNs.
- The redesigned policy preserved required list and read access while excluding the tested unnecessary operations.
- The results validate the evaluated permission decisions, rather than proving live access under every applicable AWS control.

 Lessons Learned
I learned to validate least privilege using both allowed and denied test cases. Checking access outside the intended bucket helped confirm the resource boundary, while upload and deletion tests confirmed that the redesigned policy supported a read-only purpose.

Evidence
Files are stored in `evidence/`:

- phaseD-allowed-read_bucket.png
- PhaseD_denied_outside_bucket.png
- PhaseD_group permission_updated.png
- phaseD_simulation_json.png
- PhaseD_threatmodel.pdf

 Completion Checklist
- [x] Policy redesigned using least privilege
- [x] Broad S3 managed policy removed from the lab group
- [x] Simulation confirmed intended allowed and denied results
- [x] Threat model completed

References
- [AWS IAM Policy Simulator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)
- [S3 Identity-Based Policy Examples](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-policies-s3.html)
