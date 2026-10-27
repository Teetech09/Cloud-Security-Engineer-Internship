 Week 4, Task 14: Phase A - Theory & Conceptual Overview

 Objective
Understand the purpose of AWS Secrets Manager and its role in secure credential management.

Actions Taken
- Reviewed the AWS Secrets Manager documentation covering basics, encryption and rotation.
- Studied how Secrets Manager uses envelope encryption with AWS KMS keys (AWS managed key vs customer managed key).
- Compared environment variables with managed secrets (storage, exposure, rotation, access control and audit).
- Mapped the secret lifecycle: creation, rotation, retrieval, deletion.
- Examined how IAM policies restrict access to specific secret ARNs, paired with `kms:Decrypt` on the key.
- Identified five risks of hardcoding credentials.
- Summarized encryption at rest (AES-256 via KMS) vs in transit (TLS, SigV4 signing).

Key Observation
Secrets Manager replaces static, widely copied credentials with encrypted, access-controlled, automatically rotated and audited secrets retrieved at runtime. Reading a secret requires two permissions (`secretsmanager:GetSecretValue` and KMS decrypt access), which gives defense in depth.

Evidence
- [PhaseA_Overview.pdf](./PhaseA_Overview.pdf): one-page summary document

