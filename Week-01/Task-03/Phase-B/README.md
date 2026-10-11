Phase B — Secure Environment Setup

Objective
Configure two named AWS CLI profiles with different permission levels and verify the identity associated with each profile.

Actions Taken
1. Created two IAM users:
   - `dev-user` with `ReadOnlyAccess`.
   - `ops-user` with `AdministratorAccess`, as required by the lab.
2. Generated a separate access key for each user.
3. Configured the profiles in Windows PowerShell:

```powershell
aws configure --profile dev-user
aws configure --profile ops-user
```

4. Set the default region to `us-east-1` and output format to `json`.
5. Verified each profile’s identity:

```powershell
aws sts get-caller-identity --profile dev-user
aws sts get-caller-identity --profile ops-user
```

Key Observations
Each profile returned an ARN identifying its corresponding IAM user. This confirmed that the profiles authenticated using the intended identities.

STS identity checks do not verify read-only or administrator permissions. Those permissions were assigned through the IAM policies and require separate validation.

Security Analysis
Separate profiles help distinguish development and administration activities. However, `ReadOnlyAccess` grants broad read access, and `AdministratorAccess` grants broad administrative permissions. These policies meet the lab requirements but should not be described as strict least privilege.

The profiles use long-term access keys, which require secure storage and revocation when no longer needed.

Credential Storage
AWS CLI credentials are stored locally at:

```text
%USERPROFILE%\.aws\credentials
```

The task specifies `chmod 600`, which applies to Linux/macOS. Windows file permissions must be used for this environment. Verification of those permissions is pending.

Evidence
- [IAM users screenshot](./phaseB_users.png)
- [CLI profile identity checks](./phaseB_profiles.png)

Public screenshots are redacted. Access keys, secret access keys, and the credentials file are excluded from this repository.
 Lessons Learned
Named profiles allow me to select the intended AWS identity for each command. I also learned that successful authentication does not prove the identity’s permissions and that administrator access must be treated carefully.

Completion Checklist
- [x] Created two IAM users.
- [x] Configured two named profiles.
- [x] Verified both identities using STS.

Status
Profile setup is complete; credential-file protection and permission validation remain.
