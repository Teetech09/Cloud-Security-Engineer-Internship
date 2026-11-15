Action Taken
I deployed the CloudGoat scenario iam_privesc_by_rollback with cloudgoat create. Terraform completed with 5 resources added and no errors. I configured the generated credentials for the lab user as a separate AWS CLI profile (raynor) and verified connectivity with aws sts get-caller-identity --profile raynor. I then enumerated the IAM users, roles and customer-managed policies the scenario created, and inspected the policy’s version history with aws iam list-policy-versions.

Key Observation

The scenario created one low-privilege IAM user, raynor-cgidtpp478kuxh, and one customer-managed policy, cg-raynor-policy-cgidtpp478kuxh.
No IAM roles were created. The brief asked for roles, but this scenario’s weakness is in the policy, not in role trust.
The policy has five versions (v1 to v5). Only v1 is the default (active) version. The other four versions exist but are not in effect.
That is the misconfiguration: a user who can set a policy’s default version (iam:SetDefaultPolicyVersion) can roll back to a more permissive older version and escalate privileges without creating or attaching anything new. This is the “rollback” in the scenario name.
The identity check confirmed the starting position: I begin as a low-privilege user, with the escalation path hidden in the policy history.

Evidence
PhaseB_identity.png shows get-caller-identity returning the ARN arn:aws:iam::<ACCOUNT_ID>:user/raynor-cgidtpp478kuxh, which confirms the environment is deployed and the credentials are valid.

Resources table
Resource Type	Name	ARN	Notes
IAM User	raynor-cgidtpp478kuxh	arn:aws:iam::<ACCOUNT_ID>:user/raynor-cgidtpp478kuxh	Low-privilege starting user
IAM Policy	cg-raynor-policy-cgidtpp478kuxh	arn:aws:iam::<ACCOUNT_ID>:policy/cg-raynor-policy-cgidtpp478kuxh	Customer-managed, versions v1 to v5, default = v1
IAM Roles	None	n/a	list-roles returned no roles for this scenario ID

Checklist
[x] CloudGoat environment deployed
[x] Verified user identity
[x] Documented all roles and their ARNs (none exist; user and policy documented instead)
