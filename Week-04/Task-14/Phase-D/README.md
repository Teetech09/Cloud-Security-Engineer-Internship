Phase D - Independent Extension: Secret Rotation & Threat Modeling

Objective
Implement automatic rotation for the secret and develop a threat model for credential exposure scenarios.

Actions Taken
I created a rotation function named `RotateAppDBCredentials` (Python [CONFIRM: 3.12]) from the generic rotation pattern, because Secrets Manager provides ready-made templates only for database secrets and my secret has no backing database. The function creates a new password in the AWSPENDING version and then promotes it to AWSCURRENT. Its setup and test steps do nothing, because there is no external database to update. I gave the function its own execution role with an inline policy limited to the secret's ARN (DescribeSecret, GetSecretValue, PutSecretValue and UpdateSecretVersionStage, plus GetRandomPassword), and I kept it separate from `LambdaSecretAccessRole` so that role stays read-only.

The first attempt to enable rotation failed because Secrets Manager could not invoke the function. I fixed it by adding a resource-based policy statement that allows the principal `secretsmanager.amazonaws.com` to perform `lambda:InvokeFunction`. I then enabled automatic rotation on `AppDBCredentials` with a 30-day schedule (Figure 1). [CONFIRM OR DELETE: After rotation ran, the secret showed a new AWSCURRENT version and the earlier one as AWSPREVIOUS, and `RetrieveSecretFunction` still returned `{"username": "appuser"}`.]

I also built a threat model (Figure 2) with the database credentials as the asset, five threats (unauthorized access, a misconfigured IAM role, exposure through code or logs, a stale long-lived credential, and undetected misuse), and encryption, rotation, logging and least-privilege IAM as controls. A likelihood vs impact risk matrix ranks the threats before and after those controls.

Results and Findings
Rotation worked as a mechanism, but it protects a real system only when the rotation function updates the actual service, such as a database. In this lab the stored password changes while nothing else does. The threat model rated unauthorized access and IAM misconfiguration as the highest inherent risks, and the controls reduced all five threats to Medium or Low. The unexpected finding was that rotation needed a separate invoke permission in addition to the function's role, so a correct IAM setup alone was not enough.

Key Observations
Rotation changes the impact of a leak rather than its likelihood: a copied password stops working at the next rotation, so a long-term compromise becomes a short-lived one. Keeping the rotation role separate from the retrieval role preserved least privilege, since the role that reads the secret never gained write access.

Evidence Produced
- `RotationConfig.png` (Figure 1): rotation enabled with a 30-day schedule and `RotateAppDBCredentials`.
- `PhaseD_ThreatModel.pdf` (Figure 2): threat model diagram, threat register, and risk matrix.

Completion Checklist
- Rotation enabled successfully.
- Threat model completed.
- Risk matrix documented.

## Next
Phase E
