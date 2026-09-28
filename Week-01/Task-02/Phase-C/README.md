# Phase C — Identity Hardening

## Objective
Implement IAM controls that support least privilege in my AWS lab account.

## Actions Taken
- Created IAM user groups named **Admins**, **Developers**, and **Auditors**.
- Attached `AdministratorAccess` to Admins and `ReadOnlyAccess` to Auditors.
- Created `DevelopersEncryptedIdentityLabAccess` and attached it to Developers. The custom policy permits listing the `encrypted-identity-lab` bucket and reading or uploading objects in that bucket.
- Configured the IAM account password policy using AWS CLI in CloudShell and verified it with `aws iam get-account-password-policy`.
- Checked the root user's Security credentials page. No root access keys existed, so none required deletion.

## Security Observation
The custom Developers policy scopes its S3 permissions to the dedicated lab bucket. I did not attach `PowerUserAccess` to Developers because its broader permissions would undermine that bucket restriction.

My local AWS CLI default profile pointed to a different lab account. I verified the account identity in CloudShell before running the password policy command in my own account.

## Evidence
- `IAM_Policy.json` — custom Developers policy
- `PasswordPolicy.png` — IAM password policy page
- `PasswordPolicy_CLI.png` — CLI verification of the password policy
- Group permissions screenshots — policies attached to the three groups
- Root access keys screenshot — confirms no root keys existed

## Lesson Learned
A custom policy that allows access to one bucket does not restrict access granted by another broad policy. I also learned to confirm the active AWS account before running CLI commands.
