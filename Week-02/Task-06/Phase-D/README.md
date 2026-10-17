Phase D — Permissions Boundary Implementation & Threat Modeling

Status

Boundary implementation and before-and-after simulations completed for both the initial sandbox role and a deployed IAM Vulnerable role.

Testing was limited to IAM policy simulation. No live privilege escalation was performed.

Objective

Apply a permissions boundary to restrict identity-based permissions and test its effect on a permission associated with privilege escalation.

Actions Taken

1. Reviewed the Initial Sandbox Role

I inspected `intern_role` before making changes:

- Its trust policy allows the EC2 service to assume it.
- No managed permissions policies were attached.
- Its inline policy, `DescribeInstancesOnly`, allows `ec2:DescribeInstances` on all resources.
- No permissions boundary was initially attached.

2. Created the Boundary Policy

I created `boundary_restrict_admin.json`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDataLakeListing",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example-data-lake"
    },
    {
      "Sid": "AllowDataLakeReading",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-data-lake/*"
    }
  ]
}
```

`example-data-lake` is a placeholder used for the exercise.

I created the customer-managed policy
`InternLabRestrictAdminBoundary` from this file.

Used as a boundary, this policy limits identity-based grants to the specified S3 read operations. It does not grant access independently.

3. Tested the Initial Role

Before boundary attachment, IAM simulation returned `allowed` for `ec2:DescribeInstances`.

I attached the boundary to `intern_role`, checked its configuration, and repeated the simulation.

The decision changed to `implicitDeny`, with
`AllowedByPermissionsBoundary: false`.

This demonstrated permission restriction rather than a privilege escalation scenario.

4. Reviewed the Deployed IAM Vulnerable Role

After Afzal supplied the updated repository, I deployed the Terraform environment and selected:

```text
privesc9-AttachRolePolicy-role
```

This role belongs to the `privesc-paths` module.

It had no inline policies and one attached managed policy,
`privesc9-AttachRolePolicy`, containing:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "iam:AttachRolePolicy",
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

The policy allows attaching managed policies to any role without conditions. This creates a potential privilege escalation path if an attacker can attach a powerful policy to a role they can use, subject to other applicable controls.

5. Captured the Escalation-Related Baseline

I used `aws iam simulate-principal-policy` to test
`iam:AttachRolePolicy` against the selected role itself.

Before boundary attachment, the result was `allowed`.

No managed policy was actually attached during this test.

### 6. Attached and Verified the Boundary

Using the `personal-sandbox` AWS CLI profile, I attached
`InternLabRestrictAdminBoundary` to
`privesc9-AttachRolePolicy-role`.

I verified the attachment using `aws iam get-role`.

7. Repeated the Same Simulation

After attachment, I repeated the test using the same role, action, and target resource.

The result was:

```text
Action: iam:AttachRolePolicy
Decision: implicitDeny
AllowedByPermissionsBoundary: false
```

The boundary excluded the tested IAM action, preventing the role's identity-based policy from granting it.

Results

| Role | Tested Action | Before Boundary | After Boundary |
|---|---|---|---|
| `intern_role` | `ec2:DescribeInstances` | `allowed` | `implicitDeny` |
| `privesc9-AttachRolePolicy-role` | `iam:AttachRolePolicy` | `allowed` | `implicitDeny` |

Both after-boundary simulations returned
`AllowedByPermissionsBoundary: false`.

Key Observations

- A permissions boundary limits what identity-based policies can grant; it does not grant access itself.
- An identity-policy allow cannot grant an action excluded by the boundary.
- The deployed lab role had a policy-management permission that could support privilege escalation.
- The boundary restricted that permission in simulation without changing the attached permissions policy.
- Restrictive boundaries can also block legitimate functionality if they do not match business requirements.
- Neither role gained S3 access merely because the boundary permitted S3 actions.
- Permissions to remove or modify boundaries must be controlled separately.

Threat Model

| Element | Analysis |
|---|---|
| Assets | IAM roles, their permissions, and accessible AWS resources |
| Threat actor | An attacker controlling an over-permissioned role session |
| Misconfiguration | Unconditional `iam:AttachRolePolicy` permission on all resources |
| Potential path | Attach a powerful managed policy to a role the attacker can use |
| Confidentiality impact | Potential unauthorized access to sensitive data |
| Integrity impact | Potential unauthorized modification of permissions or resources |
| Availability impact | Potential disruption or deletion of accessible resources |
| Control tested | An S3-only boundary excluding IAM policy-management actions |
| Verified result | The simulated attachment permission changed from allowed to implicitly denied |
| Residual risk | Other escalation paths and boundary modification permissions were not tested |

Resource-based grants have additional evaluation rules. These tests assessed identity-based permissions and their interaction with attached boundaries.

 Evidence

Initial Sandbox Test

- `boundary_restrict_admin.json`
- `PhaseD_Before_Boundary.png`
- `PhaseD_Boundary_Attachment.png`
- `PhaseD_After_Boundary_Denied.png`

Deployed Lab Test

- Screenshot of the `privesc9-AttachRolePolicy` policy document
- `PhaseD_Privesc_Before_Boundary.png`
- `PhaseD_Privesc_Boundary_Attached.png`
- `PhaseD_Privesc_After_Boundary.png`

## Troubleshooting

An initial IAM request failed because the CLI could not connect to the IAM endpoint.

I checked HTTPS connectivity using `Test-NetConnection` and verified authentication using AWS STS. Subsequent requests succeeded.

The earlier repository blocker was resolved when my mentor supplied the BishopFox IAM Vulnerable repository.

Testing Limitations

I simulated an escalation-related permission on a role deployed by the `privesc-paths` Terraform module.

I did not assume the role, attach an administrative policy, or execute a live escalation.

The results demonstrate that the boundary restricted the selected permission. They do not prove that all privilege escalation paths are blocked.

Lessons Learned

I learned to compare the same action and target resource before and after boundary attachment to demonstrate its effect.

I also learned to distinguish an ordinary permission restriction from an escalation-related test, and a simulation result from a live attack outcome.

Cleanup

The boundary attachment to the Terraform-created lab role was a manual change made through the AWS CLI.

The boundary policy and its attachment to `intern_role` were created outside the IAM Vulnerable Terraform deployment. They require separate cleanup consideration.

Cleanup completion will be recorded after verification.

Completion Checklist

- [x] Boundary JSON created.
- [x] Boundary attached to `intern_role`.
- [x] Initial before-and-after simulation captured.
- [x] Deployed `privesc-paths` role inspected.
- [x] Boundary attached to the selected lab role.
- [x] Escalation-related permission simulated before and after attachment.
- [x] Results and threat model documented.

Live escalation testing was not performed.

References

- [BishopFox IAM Vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Permissions Boundaries for IAM Entities](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)
- [IAM Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
