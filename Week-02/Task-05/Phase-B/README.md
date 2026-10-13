 Phase B — IAM Homelab Setup

Objective
Create dedicated IAM lab identities and configure permissions to explore group-based access, role trust relationships and least privilege.

Resources Created

| Resource | Name | Configuration |
|---|---|---|
| IAM user | `intern_user` | Added to `intern_group` |
| IAM group | `intern_group` | Attached `AmazonS3ReadOnlyAccess` |
| IAM role | `intern_role` | Trusts the EC2 service |
| Role inline policy | `DescribeInstancesOnly` | Allows `ec2:DescribeInstances` only |

Actions Taken

User and Group
I created `intern_user` and `intern_group`, then added the user to the group. I attached the AWS managed policy `AmazonS3ReadOnlyAccess` to the group so the user inherits its S3 read permissions.

Role and Trust Relationship
I created `intern_role` with EC2 as the trusted service. Its trust policy permits `ec2.amazonaws.com` to assume the role.

The role was not attached to an EC2 instance during this phase.

Custom Inline Policy
I added the following inline policy to `intern_role`, named `DescribeInstancesOnly`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    }
  ]
}
```

The resource wildcard is required because `ec2:DescribeInstances` does not support resource-level permissions.

Permission Testing
I used Principal mode in the IAM Policy Simulator to evaluate the role's inline policy.

| Action | Observed Result |
|---|---|
| `ec2:DescribeInstances` | Allowed |
| `ec2:StartInstances` | Denied |
| `ec2:TerminateInstances` | Denied |

The results confirmed that the tested read operation was allowed while the tested instance-changing operations were denied. These were simulations; no instances were started or terminated.

CLI Verification
I ran the following commands in AWS CloudShell:

```bash
aws iam get-user
aws iam get-user --user-name intern_user
aws iam get-group --group-name intern_group
aws iam list-attached-group-policies --group-name intern_group
aws iam get-role --role-name intern_role
aws iam get-role-policy \
  --role-name intern_role \
  --policy-name DescribeInstancesOnly
```

I saved the output to `phaseB_output.txt` and checked it. No errors were reported.

The command `aws iam get-user` checks the current IAM identity. The command with `--user-name intern_user` explicitly checks the lab user.
 Key Observations
- Group membership allows users to inherit the group's permissions.
- The EC2 permissions belong to `intern_role`; they are not automatically inherited by `intern_user`.
- A role's trust policy determines who can assume it, while its permissions policy determines what it can do.
- A resource wildcard is not always excessive. For this EC2 action, least privilege is achieved by restricting the allowed action.
- Dedicated IAM names organise the lab but do not create an isolation boundary.
- The S3 managed policy provides broad read access rather than access restricted to one lab bucket. A bucket-scoped policy would provide tighter access for a specific workload.
- CLI checks verified configuration, while the simulator evaluated authorization. Neither demonstrated a live EC2 role session.

Lessons Learned
I learned how users inherit group permissions and how role trust differs from role permissions. Testing allowed and denied actions helped me verify that the inline policy supported its intended purpose without granting the tested write operations.

Evidence
Evidence is stored in the `evidence/` folder:

- `phaseB-user-group-membership.png`
- `phaseB-group-created.png`
- `phaseB-group-s3-readonly.png`
- `phaseB-role-trust.png`
- `phaseB-inline-policy.json.png`
- `phaseB-inline-policy-test.png`
- `phaseB_output.txt`

 Completion Checklist
- [x] User created successfully
- [x] Group created successfully
- [x] Role created successfully
- [x] User added to group
- [x] S3 read-only managed policy attached to group
- [x] Custom inline policy created and tested
- [x] CLI configuration checks completed without errors
- [x] CLI output saved

References
- [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)
- [IAM Policy Simulator](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)
