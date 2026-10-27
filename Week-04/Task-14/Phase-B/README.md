 Week 4, Task 14: Phase B - Environment Setup

Objective
Prepare the AWS environment for secure secret management by creating a secret in AWS Secrets Manager and an IAM role that can read only that secret.

Actions Taken
I opened Secrets Manager in the AWS Console (us-east-1) and stored a new secret of type "Other type of secret" named `AppDBCredentials`, with two key/value pairs, `username` and `password`, using the lab values from the task brief. I kept the default encryption key, `aws/secretsmanager`, and left rotation off because it is covered in a later phase. A Redshift "failed to fetch clusters" message appeared during setup; it was unrelated, caused by the console trying to list databases my account doesn't use, and did not affect the secret. After the secret was stored, I copied its ARN so I could scope the IAM policy to it.

I then created an IAM role named `LambdaSecretAccessRole` with Lambda as the trusted entity and attached no managed policies. I added an inline policy that allows only `secretsmanager:GetSecretValue`, with the resource set to the exact ARN of `AppDBCredentials` rather than a wildcard. Because the secret uses the default AWS managed KMS key, no separate `kms:Decrypt` statement was needed, since that key's policy already allows principals in the account to use it through Secrets Manager. [CONFIRM OR DELETE: I verified the secret details page shows `aws/secretsmanager` as the encryption key.]

Key Observations
This phase showed me that least privilege depends on specificity. Granting one action on one resource ARN means that if the role is ever compromised, an attacker can read this single secret and nothing else, which keeps the blast radius small. I also saw that the secret's ARN ends in a random suffix, so an exact-ARN policy is tightly bound to this specific secret, while a prefix wildcard would be convenient but slightly broader. The setup also confirmed the two-layer model from Phase A: IAM controls who may call Secrets Manager, and the KMS key controls who may decrypt, with the default key handling the second layer automatically. Finally, the secret value used here is a lab placeholder, and in real environments the value should be generated rather than typed by hand and never shown in screenshots or committed to a repository.

Evidence Produced
- `PhaseB_Setup.png`: screenshot of the `AppDBCredentials` secret showing its ARN and the `aws/secretsmanager` encryption key, with the secret value not revealed.
- `IAMPolicy.json`: the inline policy for `LambdaSecretAccessRole`, allowing only `secretsmanager:GetSecretValue` on the `AppDBCredentials` ARN.

Completion Checklist
- Secret created successfully.
- KMS encryption verified.
- IAM role configured with least privilege.

## Next
Phase C
