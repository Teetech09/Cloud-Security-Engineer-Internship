 Phase A — AWS IAM Theory and Policy Simulation

 Objective
Understand IAM users, groups, roles and policies, and explore how permissions control access to AWS services.

Core Concepts

| Component | Purpose |
|---|---|
| User | An identity for a person or application with long-term credentials. |
| Group | A collection of IAM users that share permissions. Groups cannot sign in or assume roles. |
| Role | An identity assumed by a trusted principal using temporary credentials. |
| Policy | A JSON document defining permissions for actions and resources. |

 Authentication and Authorization
Authentication verifies who is making a request, using credentials such as a password, MFA or temporary security credentials.

Authorization determines whether that identity has permission to perform the requested action on a resource.

 Trust Relationships
A role's trust policy defines who can assume it, such as an AWS service or an identity in another account. Its permissions policies define what it can do after being assumed.

Practical Activity
I used the AWS IAM Policy Simulator to test a custom policy named `PhaseA-S3Wildcard-Test`.

The policy granted all S3 actions on all resources:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}

I tested the following actions:

| Action | Observed Result |
|---|---|
| s3:ListAllMyBuckets | Allowed |
| s3:GetObject | Allowed |
| s3:PutObject | Allowed |
| s3:DeleteObject | Allowed |
| s3:DeleteBucket | Allowed |
| ec2:DescribeInstances | [Enter the result shown in your simulator] |

The EC2 action was tested against the same S3 policy without adding EC2 permissions.

 Key Observations
- `s3:*` grants every S3 action, including actions beyond those tested.
- `"Resource": "*"` does not restrict permissions to a specific bucket or object.
- All five tested S3 actions were allowed, including write and delete operations.
- The expected EC2 result is an implicit deny because the policy does not allow EC2 actions.
- Simulation evaluated permissions without performing the actions against live resources.

Security Risks

| Security Principle | Potential Impact |
|---|---|
| Confidentiality | Unnecessary read permissions could expose sensitive objects. |
| Integrity | Write permissions could allow objects to be altered or overwritten. |
| Availability | Delete permissions could allow objects or buckets to be removed, subject to other applicable controls. |

Recommendation
Replace broad S3 permissions with the specific actions required for a task. Scope permissions to the necessary bucket and object ARNs wherever the actions support resource-level permissions.

Some actions, such as `s3:ListAllMyBuckets`, require `"Resource": "*"`; this should be assessed separately rather than treating every resource wildcard as a misconfiguration.

Troubleshooting
I initially selected the test actions inside the policy editor, which replaced `s3:*` with specific actions. I restored the wildcard policy, saved it, returned to the simulator and added the actions under the simulation controls.

Lessons Learned
I learned to distinguish the permissions written in a policy from the actions selected for testing. The results demonstrated that a policy can allow broad access within one service while providing no permissions for another service.

Evidence
Screenshots are stored in the `evidence/` folder:
- `s3-actions-allowed.png`
- `s3-ec2-simulation-results.png`

References
- [AWS IAM Introduction](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
- [AWS IAM Policy Simulator Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)
