Phase C: Core Execution, Enumeration & Weak Credential Testing

Objective
The objective of this phase was to enumerate the IAM entities in the lab account, identify weak credential scenarios, and analyse the account's permissions with ScoutSuite. The target was the iam-vulnerable environment deployed in Phase B.

Actions Taken
The lab instructions pointed the IAM commands at `http://localhost:4566`, which is the default LocalStack address. I did not have LocalStack running, and the vulnerable environment I deployed in Phase B lives in a real AWS lab account, so I removed the `--endpoint-url` flag and ran the same enumeration commands against that account. This kept the work on the environment that was actually built for this task.

I listed all IAM users with `aws iam list-users | jq '.Users[].UserName'`, which returned 51 users. I then looped over every user with `aws iam list-attached-user-policies` to record the managed policies attached to each one. I followed this with `aws iam list-user-policies` for users that showed no attached policies, to check for inline permissions, and I listed roles and groups with `aws iam list-roles` and `aws iam list-groups`. To look at credential hygiene, I generated and read the IAM credential report with `aws iam generate-credential-report` and `aws iam get-credential-report`. 

I then ran ScoutSuite against the account with `scout aws --profile default --report-dir ./scout_report` and reviewed the HTML report. [FILL IN: install method, for example a Python virtual environment, and anything that went wrong.]

Key Observations
The user list mixes two groups. The first is the iam-vulnerable users, named `privesc*`, `fn*`, and `fp*`, and the second is users left over from my earlier labs, such as `ops-user` and `cloudgoat-lab`. Two users, `ops-user` and `Cloud-security-lab-admin`, have the `AdministratorAccess` managed policy attached, which is the highest-risk configuration in the account because a leaked key for either one would give full control.

The user `fn1-privesc3-partial-user` has a policy named `fn1-passrole-star`, which is the over-permissive wildcard pattern studied in Phase A, where `iam:PassRole` is allowed on every resource. The user `fp2-allow-and-deny-multiple-policies-user` has both an allow-all and a deny-all policy attached. Because an explicit deny overrides an allow in IAM, this combination is a false positive, so it looks dangerous but is not exploitable. Several users, including `privesc-sre-user` and `privesc-AssumeRole-start-user`, have no attached managed policies, so their permissions come from inline policies or group membership and need a separate check.

The weak credential scenario I identified is `privesc5-CreateLoginProfile-user` (and the related `privesc6-UpdateLoginProfile-user`). These users can create or change another user's console password, so an attacker holding their keys could assign a known password and log in as that user. This turns a limited key leak into full account access. The lab instructions mention weak credentials provided in lab documentation, but iam-vulnerable does not ship preset passwords, so I documented the scenario rather than testing logins against accounts I had not set up myself.


Mitigations
The least-privilege principle from Phase A applies to each of these findings. Administrator access should be replaced with scoped roles and short-lived credentials, wildcard `PassRole` should be limited to specific role ARNs, and permissions to create or update login profiles should be restricted to a small administrative group with MFA required.

Evidence Produced
The screenshot PhaseC_enum.png shows the user enumeration and the attached-policy output. The screenshot PhaseC_report.png shows the ScoutSuite report summary. The account ID has been blurred in both.
