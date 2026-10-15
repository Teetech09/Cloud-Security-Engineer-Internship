Phase A — Theory & Conceptual Foundation

Objective

Understand least privilege and how AWS IAM policies and permissions boundaries help control access.

Activities Completed

- Reviewed the concepts of least privilege, IAM policy types, and permissions boundaries.
- Compared identity-based and resource-based policies.
- Studied how excessive IAM permissions can create privilege escalation risks.
- Created a sample identity-based policy named `readonly-s3.json` in VS Code.

Key Concepts

 Least Privilege

Least privilege means granting only the permissions required to perform a task, limited to the necessary resources.

Identity-Based and Resource-Based Policies

| Policy Type | Attached To | Purpose |
|---|---|---|
| Identity-based policy | IAM user, group, or role | Defines the actions an identity can perform on specified resources. |
| Resource-based policy | A resource, such as an S3 bucket | Defines which principals can access the resource and which actions they can perform. |

Managed policies are reusable policies maintained by AWS or the customer. Inline policies belong to a single IAM identity.

Permissions Boundaries

A permissions boundary sets the maximum permissions that identity-based policies can grant to an IAM user or role. It does not grant access by itself.

For example, an identity policy allowing S3 read and write operations cannot grant write access when the permissions boundary permits only reading. Resource-based grants have additional evaluation rules, so a boundary is not a universal restriction on every access path.

Over-Permissioned Roles

A role with unnecessary IAM permissions may allow an attacker to modify policies or access more powerful roles. Restricting permissions and protecting permissions boundaries against unauthorized changes helps reduce privilege escalation risks.

Sample Policy

The `readonly-s3.json` policy allows:

- `s3:ListBucket` on `example-lab-bucket`.
- `s3:GetObject` on objects within that bucket.

The bucket name is a placeholder for this conceptual exercise.

The policy grants no upload, delete, or policy-management permissions. However, other policies could grant additional permissions; this policy alone does not enforce a read-only limit.

Evidence

- `readonly-s3.json` — sample S3 read-only policy.
- Screenshot showing the complete JSON policy open in VS Code.

Lessons Learned

I learned that bucket-level and object-level actions require different resource ARNs. Listing objects uses the bucket ARN, while reading objects uses an ARN ending in `/*`.

I also learned that granting read-only permissions differs from enforcing a permissions boundary. A boundary limits permissions granted through identity-based policies, but does not grant access itself.

 Scope

This phase involved policy creation and conceptual learning. The sample policy was not deployed or tested in AWS.

References

- [IAM Policies and Permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)
