Phase D: Visibility and Risk Analysis

Objective
Review external-access visibility and assess IAM risks affecting
business assets.

1. Access Analyzer Setup
In the AWS Console, navigated to:

IAM → Access Analyzer → External access → Create analyzer

Configured:
- Name: week2-task07-external-access
- Region: us-east-1
- Analysis type: External access
- Zone of trust: Current account

 2. Findings Reviewed
No external-access findings were displayed at the time of review.

This result does not establish that the account is free from IAM
risks. The potential escalation path identified in Phase C involves
identities and services within the same account and falls outside
the external-access analyzer's scope.

## 3. Visibility Gap
External-access analysis alone does not provide complete visibility
into IAM permission risks.

Manual policy inspection and Policy Sentry analysis identified:
- Broad role-passing and instance-profile modification permissions.
- A role allowing all actions on all resources.
- No permissions boundaries on the three inspected scenario identities.

These are policy-review observations, not Access Analyzer findings.

4. Preliminary Risk Assessment

| ID | Risk | Business assets | Potential impact | Recommended action |
|---|---|---|---|---|---|
| R1 | Broad role-passing and instance-profile permissions | EC2 workloads and resources accessible through passed roles | A compromised Kerrigan identity could potentially obtain greater access through EC2. | Restrict PassRole to approved role ARNs, add appropriate service conditions, and limit instance-profile modification. |
| R2 | Mighty role permits all actions on all resources | Account data, infrastructure, and security controls | Misuse could expose data, modify infrastructure, or disrupt services. | Replace administrative wildcard permissions with task-specific permissions and consider an appropriate permissions boundary. |
| R3 | Reliance on external-access analysis alone | IAM monitoring and security assurance | Same-account escalation paths could remain undetected, delaying remediation. | Combine external-access analysis with identity-policy reviews, permission checks, and relevant CloudTrail monitoring. |

R1 and R2 are related components of the same potential escalation
path; they should not be treated as independent incidents.

Likelihood and impact scores require documented assumptions.
Exploitation was not performed.

 5. AI-Assisted Management Summary

 Prompt
“Summarize these IAM risks into executive-level language suitable
for management reporting: broad role-passing and instance-profile
permissions, an unrestricted administrative role, and limited
visibility from relying on external-access analysis alone.”

Summary
The lab contains a potential route by which a compromised user could
gain more powerful access through EC2. The target role has unrestricted
permissions, which could increase the consequences to data,
infrastructure, and service availability.

Although Access Analyzer displayed no external-access findings,
this does not address the same-account permission risks identified.
Management should prioritise restricting role passing, reducing
administrative permissions, and using complementary monitoring
and policy reviews.

Evidence
- ../evidence/PhaseD_access_analyzer_findings.png

 Completion Status
- External-access analyzer created.
- Findings reviewed: none displayed.
- Three preliminary risk entries documented.
- AI-assisted management summary prepared.
- risk_matrix.xlsx remains to be created and uploaded.
