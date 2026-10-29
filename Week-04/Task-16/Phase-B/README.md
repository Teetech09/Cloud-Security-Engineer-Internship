Phase B: Setup Homelab / Target Environment

 Objective

The objective of this phase was to deploy a CloudGoat scenario that simulates leaked AWS credentials, so that later phases have a realistic target. The goal was to treat the scenario's starting access keys as credentials leaked from a repository, configure them locally under a dedicated profile and confirm that they authenticate against AWS.

Actions Taken

I worked in WSL2 Ubuntu on my Windows 10 laptop, using the CloudGoat installation I had set up earlier with pipx. The task brief named the scenario `iam_privesc_by_key`, but that scenario does not exist in this CloudGoat version. After listing the available scenarios I deployed `iam_privesc_by_key_rotation`, which provides a starting IAM user whose access keys play the role of the leaked credentials. Before deploying, I found that the `iam_privesc_by_rollback` scenario from the previous task was still deployed, so I destroyed it and confirmed with `cloudgoat list deployed` that no instances remained, which kept the environment clean and avoided leaving unused IAM resources in the account. I then refreshed the CloudGoat IP whitelist and ran `cloudgoat create iam_privesc_by_key_rotation`. Terraform completed with no errors and added 17 resources, and it printed only a deprecation warning about the `managed_policy_arns` argument in the scenario's own code, which does not affect the lab.

The scenario wrote the starting credentials to its `start.txt` output file. I treated these as the leaked keys and configured them locally with `aws configure --profile cg_leak`, using us-east-1 as the region. To verify access I ran `aws sts get-caller-identity --profile cg_leak`, which returned the identity of the IAM user `manager_cgidndh53tyxvd`, confirming that the leaked keys authenticate successfully.

 Observation

This phase showed how little an attacker needs once a key is leaked. The access key and secret were enough to authenticate as the manager user from my own machine, with no password, no multi-factor prompt and no other check, and the `sts get-caller-identity` call worked without needing any specific permission. That makes it a useful first move for an attacker, because it confirms a stolen key is live and reveals the account and user it belongs to without generating much noise.

Setting up the environment also reinforced several lab hygiene practices. I used a separate CLI profile for the leaked keys so they stayed isolated from the dedicated profile that deploys the lab, I removed the leftover scenario before creating a new one, and I kept the secret key out of screenshots and documentation because this repository is public. I also noted that the starting user is named after the scenario ID, which is a reminder that identity naming can leak information about an environment. The scenario stays deployed for the next phases and will be destroyed when the task is complete so that no lab resources remain in the account.

Evidence Produced

* `phaseB_identity.png`: Screenshot of `aws sts get-caller-identity --profile cg_leak` returning the identity of the IAM user `manager_cgidndh53tyxvd`, with the account ID masked.
