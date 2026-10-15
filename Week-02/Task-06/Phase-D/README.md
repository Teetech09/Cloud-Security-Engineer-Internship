Phase D — Permission Boundary Implementation & Threat Modeling

Status

Partially completed. Boundary creation, attachment, and before-and-after IAM simulations are complete.

Testing with iam-vulnerable’s privesc-paths module remains pending Afzal’s reply with the repository link or setup files.

Objective

Apply a permissions boundary to restrict permissions available to a sandbox IAM role and evaluate its effect using IAM policy simulation.

Actions Taken
 1. Reviewed the Existing Role

I inspected `intern_role` before making changes.

- Its trust policy allows the EC2 service to assume the role.
- No managed permissions policies were attached.
- Its inline policy, `DescribeInstancesOnly`, allows `ec2:DescribeInstances` on all resources.
- No permissions boundary was initially attached.

2. Captured the Baseline

I used `aws iam simulate-principal-policy` to evaluate `ec2:DescribeInstances` for the role before attaching the boundary.

Result: `allowed`.

3. Created the Boundary Policy

I created `boundary_restrict_admin.json` with the following policy:

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

`example-data-lake` is a placeholder used for the policy exercise.

I created a customer-managed policy named
`InternLabRestrictAdminBoundary` from this file.

4. Attached the Boundary

I attached `InternLabRestrictAdminBoundary` to `intern_role` using:

aws iam put-role-permissions-boundary \
  --role-name intern_role \
  --permissions-boundary <boundary-policy-arn>

I used `aws iam get-role` to check the boundary configuration.

 5. Repeated the Simulation

After attachment, I repeated the simulation for
`ec2:DescribeInstances`.

The result changed to `implicitDeny`, with
`AllowedByPermissionsBoundary: false`.

 Results

| Test | Before Boundary | After Boundary |
|---|---|---|
| ec2:DescribeInstances | allowed | implicitDeny |

The boundary restricted an action that the role’s identity-based policy allowed. EC2 actions were excluded from the boundary’s allow list.

This was a policy simulation. No live EC2 operation or privilege escalation attempt was performed.

 Key Observations

- A permissions boundary limits what identity-based policies can grant; it does not grant access itself.
- An identity-policy allow is insufficient when the boundary does not permit the action.
- Restrictive boundaries can remove legitimate functionality if they do not match the role’s business requirements.
- The role’s only identity policy permits EC2 listing. With the S3-only boundary attached, that EC2 permission is ineffective, and the boundary does not independently provide S3 access.
- The tested role did not have administrative IAM permissions, so this exercise does not demonstrate an existing privilege escalation vulnerability.

Threat Model

| Element | Analysis |
|---|---|
| Asset | AWS resources accessible through the sandbox role |
| Threat | Misuse of excessive permissions following role compromise or an incorrect policy change |
| Potential path | Additional identity-based permissions could expand access without an effective permission limit |
| Control | A boundary restricts identity-based grants to specified S3 read actions and resources |
| Verified result | EC2 listing changed from allowed to implicitly denied in simulation |
| Limitation | Administrative escalation paths and the privesc-paths module were not tested |
| Control dependency | Permissions to change or remove the boundary must also be controlled |

Resource-based grants have additional evaluation rules. This exercise evaluated the role’s identity-based permission and its attached boundary.

 Evidence

- `boundary_restrict_admin.json`
- `PhaseD_Before_Boundary.png`
- `PhaseD_Boundary_Attachment.png`
- `PhaseD_After_Boundary_Denied.png`

 Blocker and Remaining Work

The task brief did not include the iam-vulnerable repository link or setup files. These have been requested from Afzal.

Pending activities:

- Deploy the intended lab environment.
- Run the privesc-paths module.
- Record the actual attempts and outcomes.
- Assess which outcomes can be attributed to the boundary.

No blocked escalation results are claimed at this stage.

 Lessons Learned

I observed how a permissions boundary changes effective permissions even when the original identity policy remains unchanged.

I also learned to distinguish a denied permission simulation from a blocked privilege escalation attempt, and to document incomplete testing accurately.

Completion Checklist

- [x] Boundary JSON created.
- [x] Boundary attached to intern_role.
- [x] Before-and-after simulation captured.
- [ ] privesc-paths module executed.
- [ ] Escalation outcomes documented.

 References

- [Permissions Boundaries for IAM Entities](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)
- [IAM Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
