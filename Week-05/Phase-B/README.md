Phase B: Environment Setup

Objective

The objective of this phase was to deploy the CloudGoat `ec2_ssrf` scenario, which simulates the compromise of an EC2 instance role, and to record the key identifiers of the deployed resources so they can be used in the attack and detection phases that follow.

Actions Taken

I ran the deployment from the Ubuntu distribution in WSL2 using `cloudgoat create ec2_ssrf`, with the `cloudgoat` AWS profile in `us-east-1`. My first command attempt ran from PowerShell, where CloudGoat is not installed, and a plain `wsl` session opened the Docker Desktop internal distribution because it is the default. I resolved both by opening the Ubuntu distribution explicitly with `wsl -d Ubuntu -u teetech`.

The first deployment failed at the final Terraform step, where a file provisioner connects to the new instance over SSH. The error was a timeout on port 22, which pointed to the security group rejecting my connection because my public IP no longer matched the CloudGoat whitelist. I refreshed the whitelist with `cloudgoat config whitelist --auto`, destroyed the partial deployment with `cloudgoat destroy ec2_ssrf` so it would not keep running, and redeployed. The second run completed successfully with 27 resources added.

After deployment, I recorded the identifiers of the resources I would be working with, using `aws iam list-roles` and `aws ec2 describe-instances`. The scenario ID is `cgidzu8n53qwm5` and the EC2 instance ID is `i-043033d482b3dc12a`. The instance uses the role `cg-ec2-role-cgidzu8n53qwm5`, with the ARN `arn:aws:iam::718543338561:role/cg-ec2-role-cgidzu8n53qwm5`, attached through the instance profile `cg-ec2-instance-profile-cgidzu8n53qwm5`. The scenario also created a second role for a Lambda function, `cg-lambda-role-cgidzu8n53qwm5-service-role`. The instance metadata option `HttpTokens` is set to `optional`.

I then tried to enable GuardDuty and Security Hub in `us-east-1` with `aws guardduty create-detector` and `aws securityhub enable-security-hub`. Both returned a `SubscriptionRequiredException`. The AWS console showed that the account is on the Free account plan, which restricts these services until the account is upgraded to the Paid plan. I chose not to upgrade the account, so neither service was enabled.

Key Observations

The `HttpTokens` value of `optional` is the most important finding of this phase. It means the instance still accepts IMDSv1 requests, so any process on the instance that can make an HTTP request, including a web application with a server-side request forgery flaw, can retrieve the temporary credentials of `cg-ec2-role-cgidzu8n53qwm5` without a token. This is the vulnerability identified in the Phase A threat model, and it confirms that the deployed environment matches the attack path I modeled, with the EC2 instance as the asset and the instance role as the credential at risk.

The scenario deployed correctly on the second attempt, and the failure on the first attempt was useful because it showed how tightly CloudGoat ties its security groups to a whitelisted IP address. Any change of public IP, which is common on mobile and shared connections, breaks provisioning until the whitelist is refreshed. It also confirmed that a failed Terraform run leaves live resources behind, so the partial deployment had to be destroyed before retrying to avoid cost and name conflicts.

The GuardDuty and Security Hub limitation is a good example of a platform constraint affecting security architecture. The error was `SubscriptionRequiredException`, not an access denied error, which showed that the problem was the account plan and not the IAM permissions of my lab user. The practical effect is that this environment has no managed detection layer, so any attack I run in later phases will not produce findings for me to review. CloudTrail remains available as the main source of evidence in its place.

 Evidence Produced

The evidence for this phase is PhaseB_resources.png, a screenshot of the deployment output showing `Apply complete! Resources: 27 added`, the instance details with `HttpTokens` set to `optional`, and the role ARN. The GuardDuty and Security Hub errors and the console message about the Free account plan are recorded as limitations in this README. The generated `solus` access keys were intentionally left out of all evidence.
