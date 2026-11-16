Phase B: Safe AWS Sandbox & Tooling Setup

Actions Taken

I prepared a controlled workstation for the flaws.cloud assessment using WSL2 Ubuntu on Windows 10. Because the default WSL distribution on my machine is Docker Desktop's internal one, I opened Ubuntu explicitly with `wsl -d Ubuntu -u teetech`. I created the working directory `~/flaws_capstone` with a single `Evidence` folder inside it, and started logging the session with `script`, so that every command and its output was captured in `Evidence/PhaseB_SetupLog.txt`.

I then verified the required tooling. AWS CLI v2 (version 2.38.4), Git, jq, curl, dig, nslookup and host were all already installed, so no installation was needed. I confirmed each one by running its version command.

For the sandbox identity, I did not replace or modify my default AWS CLI profile. I copied the credentials of an existing non-root IAM user in my dedicated training account into a new named profile called `flaws-sandbox`, set to the `us-west-2` region with JSON output. I did this with `aws configure set` and command substitution, so the keys were never typed or displayed on screen. I confirmed the identity with `aws sts get-caller-identity --profile flaws-sandbox`, and the returned ARN was an IAM user rather than the root account.

I then created a low-risk profile for Level 2. Using the sandbox profile, I created an IAM user named `flaws-l2-user` with no policies attached, generated an access key for it, and stored the key in a separate profile called `flaws-l2`. I captured the key into shell variables and cleared them afterwards, so the secret did not appear in the terminal or in the log. I verified the profile with `get-caller-identity`, which returned the `flaws-l2-user` ARN, and I confirmed with `list-attached-user-policies` that the user has no attached permissions. This gives me a valid authenticated AWS principal for Level 2 without granting any privileges.

I set up a budget alert in AWS Budgets as the billing guardrail, and I prepared a Level 4 cleanup plan covering the EC2 instance, EBS volume, security group and key pair, together with the commands I will use to remove them and to prove they are gone. All resources I create for Level 4 will be tagged `Project=flaws-capstone` so that they can be found and deleted reliably. Finally, I organised all evidence in the single `Evidence` folder with clear Phase-based filenames.

Key Observations

The most useful lesson in this phase was how easily the environment can be misconfigured before any testing starts. My first attempt at opening WSL landed in Docker Desktop's internal distribution, which is read-only, so my directory creation and logging failed. Switching to the Ubuntu distribution explicitly fixed this, and I now use that command for every session in this capstone.

I also learned to be careful with credentials when logging a terminal session. Because `script` records everything shown on screen, running `aws configure` interactively would have written my access keys into the evidence file. Copying keys between profiles with `aws configure set` and capturing new keys into variables kept them out of the log. A typing mistake in my first attempt at creating the Level 2 key left the `flaws-l2` profile with empty credentials, which produced an `InvalidClientTokenId` error. This showed me that a failed key-creation step needs to be checked and repeated, rather than assumed to have worked. After recreating the key, the profile authenticated correctly.

Keeping the Level 2 user free of permissions demonstrates least privilege in practice: the user can identify itself with STS but cannot do anything else in my account. Using separate named profiles for the sandbox and the Level 2 identity keeps the two trust levels clearly apart and leaves my default profile untouched. Planning the cleanup before creating any Level 4 resources, and tagging everything for later discovery, reduces the risk of leftover EC2 or EBS resources causing unexpected charges.

Account identifiers appear in several command outputs, so I will redact them before submitting or publishing the evidence.


Evidence Produced

All evidence is stored in the single `Evidence` folder.

- PhaseB_SetupLog.txt: the terminal session log recorded with `script`, covering the tool version checks, profile creation and identity verification. Account identifiers are redacted and the file was checked to confirm it contains no access keys.
- 
- PhaseB_Identity_redacted.png: screenshot of `aws sts get-caller-identity` for the sandbox profile and the Level 2 profile, with the account ID masked, showing IAM user identities and not the root account.
- PhaseB_Budget.png: screenshot of the AWS Budgets page showing the billing alert configured as a cost guardrail.
- PhaseB_Level4_CleanupPlan.png the documented cleanup plan for Level 4 temporary resources, with deletion order, commands and verification checks.
