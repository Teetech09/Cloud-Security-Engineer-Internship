 Phase A: Theory and IAM Enumeration

Objective
Understand IAM entity relationships, enumeration fundamentals, and how
attackers identify potential privilege escalation paths.

Activities Completed
- Reviewed IAM users, groups, roles, trust policies, and permissions policies.
- Enumerated IAM users, customer-managed policies, and roles.
- Inspected the trust policy and permissions boundary of intern_role.
- Reviewed explicit allow, implicit deny, and explicit deny.

Commands Executed

```bash
aws iam list-users --output table --no-cli-pager

aws iam list-policies --scope Local --output table --no-cli-pager

aws iam list-roles --output table --no-cli-pager

aws iam get-role --role-name intern_role --output json --no-cli-pager


Key Observations
- The account contained administrative, development, and lab-related users.
- Customer managed policies included names associated with privilege
  escalation exercises. Names alone did not confirm exploitable permissions.
- intern_role trusted ec2.amazonaws.com through sts:AssumeRole.
- InternLabRestrictAdminBoundary was applied as the role's permissions boundary.
- No role last-used information was returned. This did not prove the role
  had never been used.
- The get-role output showed the trust policy and boundary reference,
  but did not establish the role's effective permissions.

Explicit and Implicit Access Decisions
Decision	Meaning
Explicit allow	A matching Allow statement grants permission, subject to applicable policy controls.
Implicit deny	Applicable policies do not provide the required permission.
Explicit deny	A matching Deny statement blocks access and overrides an Allow.
Permissions boundaries limit permissions; they do not grant them.


Security Relevance
Enumeration reveals identities and policy relationships that require further
review. Attackers may look for excessive policy-management permissions,
unsafe trust relationships, or the ability to pass powerful roles to services.
The commands used were read-only. No exploitable privilege escalation path
was confirmed from these outputs alone.

Evidence
- ../evidence/phaseA_list_users.png
- ../evidence/phaseA_list_policies.png
- ../evidence/phaseA_get_role.png
- ../phaseA_summary.txt

Outcome
Completed IAM enumeration and reviewed the distinction between role trust,
permissions policies, and permissions boundaries.
