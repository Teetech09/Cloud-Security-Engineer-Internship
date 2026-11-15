Week 07 – Phase E: Hosted Labs Integration

Objective
Reinforce learning through external labs simulating real-world IAM exploitation.

Lab 1: PwnedLabs — Assume Privileged Role with External ID

Status: Incomplete — flag not captured**

Action Taken
Followed the lab's attack path through several stages successfully:
1. Enumerated the target with `ffuf`, discovered an exposed `config.json` containing AWS credentials for IAM user `data-bot`.
2. Confirmed identity with `aws sts get-caller-identity`.
3. Used `aws-enumerator` to identify `secretsmanager:ListSecrets` permission on the `data-bot` user.
4. Listed Secrets Manager secrets and successfully retrieved `ext/cost-optimization`, obtaining console credentials for a second IAM user, `ext-cost-user`.
5. Logged into the AWS Console as `ext-cost-user` and used browser DevTools (Network tab, filtering for `creds`) to extract service-scoped temporary credentials from the IAM service's console page, as the lab's walkthrough describes.

Where It Broke Down
Every attempt to use the DevTools-extracted temporary credentials (`accessKeyId`, `secretAccessKey`, `sessionToken`) with the AWS CLI failed, returning either `InvalidClientTokenId` or `ExpiredToken`, despite:
- Verifying the credentials parsed as valid JSON with no corrupted or truncated characters (confirmed via byte-length comparison between source and stored values).
- Confirming no stale environment variables or AWS profile conflicts were overriding the intended profile.
- Confirming system time was within the token's stated expiration window.
- Re-capturing fresh credentials multiple times and minimizing the delay between extraction and use.

One attempt did return `ExpiredToken` specifically (rather than `InvalidClientTokenId`), confirming the credential-setting mechanism itself worked correctly and that at least some of the failures were genuine timing issues — these console-federation tokens appear to be very short-lived or possibly single-use per page load, making the manual copy/extract/test workflow unreliable within normal typing/pasting speed.

Key Observation
The lab clearly demonstrates a real and serious misconfiguration chain (exposed config file → low-privilege user → Secrets Manager pivot → second user → console session credential extraction), and I was able to reproduce every step up to the final credential validation. The blocker was practical/tooling (credential freshness and transfer reliability) rather than a gap in understanding the attack path itself.

Evidence
PhaseE_lab1_progress.png — screenshots covering the successful steps (config.json discovery, data-bot identity, secret retrieval for ext-cost-user).

Next Steps (if revisited)
- Automate the credential extraction and testing into a single script or browser extension to eliminate the manual transfer delay.
- Investigate whether the lab environment issues shorter-lived tokens than typical AWS STS federation tokens.

Lab 2: Cybr — IAM Challenge Escape Room

Status: Not attempted — premium lab

This lab requires a Cybr premium subscription, which isn't available. Noting this as a platform access limitation rather than an incomplete attempt.

Checklist
- [ ] Lab 1 flag captured (enumeration and pivot completed; final privilege escalation step incomplete)
- [ ] Lab 2 completed (premium access not available)

Checklist
- [ ] Lab 1 flag captured (enumeration and pivot completed; final privilege escalation step incomplete)
- [ ] Lab 2 completed
