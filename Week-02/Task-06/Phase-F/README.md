Phase F — Documentation and Reporting

Objective

Consolidate the methodology, policy files, test results, findings, limitations, and supporting evidence from Week 2, Task 06.

Documentation Summary

Phase A — Theory and Conceptual Foundation

Studied least privilege, identity-based and resource-based policies, managed policies, and permissions boundaries.

Created a sample S3 read-only policy in VS Code.

Phase B — Environment Setup

Following the Mentor's updated instructions, deployed the BishopFox IAM Vulnerable environment using Terraform instead of the originally described Docker environment.

Deployment completed successfully:

- 265 resources added.
- 0 resources changed.
- 0 resources destroyed.

Verified the deployed IAM roles using the AWS CLI.

Documented setup challenges, including an incorrect Terraform download, configuration formatting, and an initial AWS profile mismatch. The profile mismatch was corrected before successful deployment into the dedicated sandbox account.

Phase C — Policy Engineering

Created a Data Analyst policy granting only bucket listing and object reading within the specified example data lake.

IAM simulations produced the following results:

| Test | Result |
|---|---|
| List the specified bucket | allowed |
| Read an object within the bucket | allowed |
| Upload an object | implicitDeny |
| Delete an object | implicitDeny |
| Read an object outside the bucket | implicitDeny |

AWS IAM Access Analyzer returned:

    {"findings": []}

Access Analyzer was used as an alternative because the updated repository does not provide the analyzer described in the original task. Mentor acceptance of this substitution has not yet been recorded.

 Phase D — Permissions Boundary Implementation

Created `boundary_restrict_admin.json` and the customer-managed policy `InternLabRestrictAdminBoundary`.

Tested the boundary against two roles:

| Role | Action | Before Boundary | After Boundary |
|---|---|---|---|
| intern_role | ec2:DescribeInstances | allowed | implicitDeny |
| privesc9-AttachRolePolicy-role | iam:AttachRolePolicy | allowed | implicitDeny |

Both after-boundary results showed `AllowedByPermissionsBoundary: false`.

Documented the potential escalation path, security implications, mitigation, and testing limitations.

### Phase E — Hosted Labs

The required TryHackMe and Cybr labs were inaccessible because they require premium subscriptions.

The Mentor approved skipping these labs. No completion screenshots or certificates are claimed.

 Key Findings

| Finding | Evidence | Security Significance |
|---|---|---|
| Unrestricted policy-attachment permission | The deployed lab policy allowed iam:AttachRolePolicy on all resources | Could support privilege escalation through attaching powerful policies to usable roles |
| Boundary restricted the tested IAM action | Simulation changed from allowed to implicitDeny | Demonstrated restriction of an escalation-related permission |
| Data Analyst policy matched the tested requirements | Read actions allowed; write, delete, and outside-bucket reads implicitly denied | Demonstrated scoped read access in the supplied policy |
| AWS profile mismatch during setup | Initial attempt used hosted-lab credentials | Highlighted the importance of explicit profile selection and account verification |

## Evidence Organization

Supporting evidence is recorded in the relevant phase documentation:

- Phase A: sample JSON policy and VS Code screenshot.
- Phase B: Terraform deployment confirmation and deployed role list.
- Phase C: custom policy, simulation screenshots, and Access Analyzer validation log.
- Phase D: boundary policy, policy inspection, attachment confirmations, and before-and-after simulations.
- Phase E: premium-access limitation and Mentor-approved exclusion.

Only evidence actually captured is included.

Scope and Limitations

- The data lake bucket name was a placeholder for policy simulation.
- Simulation did not read, upload, or delete actual S3 objects.
- No administrative policy was attached during escalation testing.
- No live privilege escalation was performed.
- Zero Access Analyzer validation findings do not prove the absence of every excessive-permission risk.
- The boundary tests covered selected actions rather than all possible escalation paths.

Recommendations

- Scope IAM actions and resources to documented business requirements.
- Restrict policy-management permissions such as iam:AttachRolePolicy.
- Protect permissions boundaries against unauthorized modification or removal.
- Use named AWS profiles and verify account identity before Terraform operations.
- Combine policy validation with allowed-action and denied-action testing.
- Retain Terraform state securely for cleanup and exclude secrets from GitHub.

Cleanup Status

Cleanup has not yet been verified.

The IAM Vulnerable deployment must be removed using its Terraform configuration and state after testing.

The separately created boundary policy and its attachment to `intern_role` require separate review because they are outside the lab's Terraform deployment.

Deliverables

- Phase-specific GitHub README documentation.
- Sample and custom IAM policy JSON files.
- Permissions boundary JSON file.
- Simulation and validation evidence.
- Threat model and findings.
- Documented blockers, approved exclusions, and cleanup status.

The final written report will be prepared manually and is not included in this README.

Lessons Learned

This task strengthened my understanding of least privilege, resource scoping, and the interaction between identity-based permissions and permissions boundaries.

It also reinforced the need to verify AWS account identity before deployment, distinguish simulation results from live outcomes, and document limitations accurately.
