Phase C — IAM Enumeration and Policy Analysis

Objective
Enumerate IAM identities and policies, analyse their relationships, and identify potentially excessive permissions.

Actions Taken
I used AWS CloudShell to enumerate users, groups and customer-managed policies, then inspected the lab role and relevant S3 policies.

```bash
aws iam list-users --no-cli-pager
aws iam list-groups --no-cli-pager
aws iam list-policies --scope Local --no-cli-pager
aws iam get-role --role-name intern_role --no-cli-pager
aws iam list-attached-group-policies --group-name intern_group --no-cli-pager
```

I retrieved the role’s inline policy:

```bash
aws iam get-role-policy \
  --role-name intern_role \
  --policy-name DescribeInstancesOnly \
  --no-cli-pager > policy_analysis.json
```

I retrieved version `v3` of the S3 managed policy:

```bash
aws iam get-policy-version \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --version-id v3 \
  --query 'PolicyVersion.Document' \
  --output json \
  --no-cli-pager > s3_readonly_policy.json
```

I also retrieved the customer-managed policy and checked its attachment and group membership:

```bash
POLICY_ARN=$(aws iam list-policies \
  --scope Local \
  --query "Policies[?PolicyName=='DevelopersEncryptedIdentityLabAccess'].Arn | [0]" \
  --output text \
  --no-cli-pager)

aws iam get-policy-version \
  --policy-arn "$POLICY_ARN" \
  --version-id v1 \
  --query 'PolicyVersion.Document' \
  --output json \
  --no-cli-pager > developers_policy.json

aws iam list-entities-for-policy \
  --policy-arn "$POLICY_ARN" \
  --no-cli-pager

aws iam get-group --group-name Developer --no-cli-pager
```

Findings and Recommendations

| Entity | Policy Name | Risk Level | Recommendation |
|---|---|---|---|
| `intern_group` → `intern_user` | `AmazonS3ReadOnlyAccess` | Medium, assuming only lab-bucket access is required | Replace broad read access with specific actions scoped to the required bucket and objects. |
| `intern_role` | `DescribeInstancesOnly` | Low | Retain if instance discovery is required. Its resource wildcard is necessary for this action. |
| `Developer` group — no members | `DevelopersEncryptedIdentityLabAccess` | Low; no current user access through this group | Retain bucket-scoped access and confirm upload requirements before adding members. |

 Broad S3 Read Permissions
`AmazonS3ReadOnlyAccess` permits broad Get, List and Describe operations across S3 resources, including S3 Object Lambda Get and List operations.

For a user who only needs one lab bucket, this exceeds the assumed access requirement. The main potential impact is confidentiality: an identity could retrieve sensitive objects wherever other applicable controls permit access. No data exposure was demonstrated.

EC2 Describe Permissions
`DescribeInstancesOnly` permits only `ec2:DescribeInstances`. The `"Resource": "*"` value is required because the action does not support resource-level permissions.

The Phase B simulation allowed `DescribeInstances` and denied `StartInstances` and `TerminateInstances`. No new simulation was performed during this review.

Bucket-Scoped Developer Permissions
`DevelopersEncryptedIdentityLabAccess` permits:
- `s3:ListBucket` on `encrypted-identity-lab`.
- `s3:GetObject` and `s3:PutObject` on objects within that bucket.

This policy does not itself grant deletion or policy-management permissions. Upload permission can allow writing to existing object keys, so it should match the intended task.

The policy is attached to `Developer`, which currently has no users. It therefore provides no current user access through that group.

AI-Assisted Review
I used ChatGPT to review the retrieved policy JSON for excessive privileges and safer alternatives, following the task’s review request:

> Review this JSON policy for excessive privileges and suggest safer alternatives.

The review distinguished broad S3 read permissions from a required EC2 resource wildcard. It also identified the developer policy as bucket-scoped and considered the empty group when assessing current access.

AI feedback supported the analysis; it did not verify live access or replace review of the AWS configuration.

Proposed Improvement
If `intern_user` only needs to list and read objects in `encrypted-identity-lab`, a narrower alternative is:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::encrypted-identity-lab"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::encrypted-identity-lab/*"
    }
  ]
}
```

This was a recommendation only. I did not replace the assigned managed policy during Phase C.

## Troubleshooting
I initially entered `list groups` instead of `list-groups`, which caused a CLI operation validation error. Using the hyphenated command resolved it.

A policy retrieval also failed because `VERSION_ID` was entered as a literal placeholder. I checked the policy metadata, obtained `v3`, and successfully retrieved its JSON.

Key Observations
- Customer-managed policy enumeration does not include inline or AWS managed policies.
- A resource wildcard must be assessed against the action’s resource-level support.
- Read-only permissions can still create confidentiality risks.
- Policy attachments and group membership determine who inherits group permissions.
- An empty group can have a policy attached without currently granting access to any users.
- The Phase A wildcard S3 policy was a simulator example, not an attached account finding.
- This review did not establish every identity’s complete effective permissions.

## Lessons Learned
I learned to assess actions, resource scope, policy attachments and group membership together. I also learned to distinguish a potential least-privilege issue from confirmed data exposure and to resolve CLI syntax errors without changing IAM permissions.

Evidence
Files are stored in `evidence/`:

- `policy_analysis.json`
- `s3_readonly_policy.json`
- `developers_policy.json`
- `phaseC-ai-policy-review.png`

Completion Checklist
- [x] Required user and group enumeration performed
- [x] Customer-managed policies enumerated
- [x] Lab role and relevant policy documents inspected
- [x] Potentially excessive S3 read policy identified
- [x] AI-assisted review completed
- [x] Findings and recommendations documented

References
- [AWS IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
- [S3 Identity-Based Policy Examples](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-policies-s3.html)
- [EC2 Actions and Resource Support](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2.html)
