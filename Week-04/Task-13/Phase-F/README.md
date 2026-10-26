Week 4: Secrets Exposure & Credential Lifecycle (Task 13)

Hands-on assessment of how secrets are created, stored, exposed, and revoked in AWS.

| Phase | Focus | Evidence |
|---|---|---|
| A | IAM user keys vs STS tokens vs instance roles | phaseA_identity.png |
| B | CloudGoat `iam_privesc_by_rollback` setup | phaseB_setup.png |
| C | Secret discovery with TruffleHog and GitLeaks on SecLists | phaseC_summary.png |
| D | Secret lifecycle analysis and mitigations | Phase-D/README.md |
| E | PwnedLabs: Hunt for Secrets in Git Repos; Uncover Secrets in CodeCommit and Docker | phaseE_lab1.png, phaseE_lab2.png |
| F | Executive report | Phase-F/ |

Key takeaway
Deleting a secret from the latest commit does not remove it from history or image layers, and without rotation it stays valid.

> All testing was done in lab and training environments. Credentials shown in evidence are redacted.
