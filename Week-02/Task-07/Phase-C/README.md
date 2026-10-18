Phase C: IAM Enumeration and Permission Mapping

Objective
Enumerate IAM identities and map the CloudGoat scenario's permissions
and trust relationships manually and programmatically.

Scope
Account-wide users, groups, customer managed policies, and roles were
enumerated. Detailed permission mapping focused on the CloudGoat
iam_privesc_by_attachment scenario.

1. Manual IAM Inventory

Executed the following commands:

```bash
aws iam list-users --output table --no-cli-pager
aws iam list-groups --output table --no-cli-pager
aws iam list-policies --scope Local --output table --no-cli-pager
aws iam list-roles --output table --no-cli-pager
```

The inventory included resources from previous labs. Seven IAM groups
were listed; their existence alone did not establish a relationship
with the CloudGoat user.

2. Kerrigan Permission Mapping

```bash
aws iam list-groups-for-user --user-name kerrigan --no-cli-pager
aws iam list-attached-user-policies --user-name kerrigan --no-cli-pager
aws iam list-user-policies --user-name kerrigan --no-cli-pager
```

Results:
- No group memberships.
- One attached managed policy: cg-kerrigan-policy.
- No inline user policies.
- No permissions boundary.

The policy's default version was v1. Its document was retrieved using
get-policy-version.

Key Permissions

| Category | Actions |
|---|---|
| IAM discovery | iam:ListRoles, iam:ListInstanceProfiles |
| Instance-profile modification | iam:AddRoleToInstanceProfile, iam:RemoveRoleFromInstanceProfile |
| Role passing | iam:PassRole |
| EC2 execution and association | ec2:RunInstances, ec2:AssociateIamInstanceProfile |
| Supporting EC2 permissions | ec2:CreateKeyPair and several Describe actions |

Both Allow statements used Resource: "*" and contained no conditions.

3. Role Trust Relationships

Retrieved each role's trust policy:

```bash
aws iam get-role \
  --role-name cg-ec2-meek-role-cgidj957m3fp3j \
  --query 'Role.AssumeRolePolicyDocument' \
  --output json --no-cli-pager

aws iam get-role \
  --role-name cg-ec2-mighty-role-cgidj957m3fp3j \
  --query 'Role.AssumeRolePolicyDocument' \
  --output json --no-cli-pager
```

Both roles contained the following trust statement:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

This trust permits EC2 to assume the roles. It does not directly permit
Kerrigan to assume them. EC2 service trust is expected for EC2 roles;
the risk arises from its combination with excessive role-passing and
instance-profile permissions.

4. Role Permissions

Used list-attached-role-policies and list-role-policies for both roles,
then retrieved each managed policy's default version and document.

| Role | Attached policy | Policy effect | Inline policies | Boundary |
|---|---|---|---|---|
| Meek | cg-ec2-meek-policy | Deny all actions on all resources | None | None |
| Mighty | cg-ec2-mighty-policy | Allow all actions on all resources | None | None |

The Meek role's explicit deny applies to its own sessions. It does not
restrict a separate session using the Mighty role.

Mighty's wildcard Allow grants administrative permissions at the policy
level, subject to other applicable controls.

5. Policy Sentry Analysis

Installed Policy Sentry 0.15.2 inside the Python virtual environment.

The task brief's analyze CLI command was unavailable in this version.
Inspection confirmed that the Python analysis API was available, so
analyze_by_access_level was used instead.

Exported policy documents:
- policies.json — Kerrigan policy.
- meek_policy.json — Meek role policy.
- mighty_policy.json — Mighty role policy.

The analysis classified Permissions management actions from Allow
statements only. This prevented Meek's wildcard Deny from being
misreported as granted access.

Results

| Policy | Permissions-management actions recognised |
|---|---:|
| cg-kerrigan-policy | 3 |
| cg-ec2-meek-policy | 0 |
| cg-ec2-mighty-policy | 535 |

Kerrigan's identified actions:
- iam:AddRoleToInstanceProfile
- iam:PassRole
- iam:RemoveRoleFromInstanceProfile

The count of 535 reflects Policy Sentry's installed action dataset.
It is not the total number of permissions granted by Mighty's policy.

The generated report was saved as analysis.txt.

6. Steampipe Verification

Installed Steampipe v2.4.7 and its AWS plugin. Working files were placed
under /tmp to conserve CloudShell home-directory storage.

Executed the account-wide role query:

```sql
select name, arn from aws_iam_role;
```

Then verified the scenario roles:

```bash
steampipe query "select name, arn from aws_iam_role where name like '%cgidj957m3fp3j%';"
```

The query returned:
- cg-ec2-meek-role-cgidj957m3fp3j
- cg-ec2-mighty-role-cgidj957m3fp3j

These results matched the AWS CLI inventory.

Key Finding

Kerrigan's policy combines broad role-passing, instance-profile
modification, and EC2 permissions. The Mighty role trusts EC2 and has
an all-actions, all-resources Allow policy.

Together, these relationships indicate a potential privilege escalation
path through EC2 instance profiles. No permissions boundaries were
attached to the three inspected identities.

Limitations
- Exploitation was not performed.
- Organisation policies and other applicable restrictions were not assessed.
- Policy Sentry classified actions; it did not calculate effective permissions.
- Detailed permission mapping covered the scenario identities, not every
  identity from earlier labs.

## Evidence
- ../evidence/PhaseC_list_users.png
- ../evidence/PhaseC_list_groups.png
- ../evidence/PhaseC_list_policies.png
- ../evidence/PhaseC_kerrigan_policy_mapping.png
- ../evidence/PhaseC_kerrigan_policy_document.png
- ../evidence/PhaseC_role_trust_policies.png
- ../evidence/PhaseC_role_policy_mapping.png
- ../evidence/PhaseC_meek_policy.png
- ../evidence/PhaseC_mighty_policy.png
- ../evidence/PhaseC_policy_sentry_commands.png
- ../evidence/PhaseC_policy_sentry_analysis.png
- ../evidence/PhaseC_steampipe_roles.png
- ../analysis.txt
