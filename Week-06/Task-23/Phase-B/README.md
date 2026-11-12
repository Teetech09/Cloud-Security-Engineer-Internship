Phase B: Environment Setup (CloudGoat Scenario)

Cloud Incident Response Methodology for AWS Environments

Objective

The objective of this phase was to deploy a controlled, intentionally vulnerable AWS environment that can be used to simulate a compromise in the later phases of the incident response exercise. The scenario chosen was the CloudGoat iam_privesc_by_attachment scenario, deployed in us-east-1. The phase also required enabling GuardDuty in the same region as the detection source.

Actions Taken

I worked in WSL2 Ubuntu. The brief’s instructions assume the older ./cloudgoat.py script, but the current CloudGoat release is installed as a package and run with the cloudgoat command, so I used that command instead. The fresh clone I made contained no cloudgoat.py, which is how I found this out. I confirmed the AWS profile with aws sts get-caller-identity and checked with cloudgoat list deployed that no earlier scenario remained in the account.

CloudGoat’s automatic whitelist step, which looks up my public IP through an external service, was cancelled when that request did not complete. I got my public IPv4 address from another service and wrote it to the CloudGoat whitelist.txt file by hand, so that the scenario’s resources would accept connections only from my address. I then deployed the scenario with cloudgoat create iam_privesc_by_attachment --profile cloudgoat. Terraform completed without errors and reported 16 resources added. The scenario received the ID cgidjij1nc9chc, and the starting credentials for the user Kerrigan were written to the scenario’s start.txt file, which I redacted before taking my evidence.

I recorded the created resources with terraform state list and with AWS CLI queries filtered by the scenario ID. These showed two IAM roles, cg-ec2-meek-role-cgidjij1nc9chc and cg-ec2-mighty-role-cgidjij1nc9chc, and a running t3.micro EC2 instance, i-0d0b8f14ddab8eec2, tagged super-critical-security-server. [Add the instance profile name and the other resources from your terraform state list output, such as the user, VPC, subnet and security group.] A stopped EC2 instance from earlier Week-01 work also appeared in the account. It is not part of this scenario, so I excluded it from my records.

I then tried to enable GuardDuty with aws guardduty create-detector --enable in us-east-1. The command returned a SubscriptionRequiredException, and list-detectors returned the same error.

Key Observations

The scenario deployed successfully and gives a controlled target for the next phases: a starting IAM user, two EC2 roles with different privilege levels and an EC2 instance. Restricting the whitelist to my own IP address limits who can reach the scenario’s resources while it stays deployed.

GuardDuty could not be enabled because my AWS account is on the Free account plan and the service requires a subscription. I decided not to upgrade the account for this exercise, so the “GuardDuty enabled and verified” checklist item is not met, and I have recorded it as a limitation and not as a pass. This changes how the later phases work. Detection and analysis will rely on CloudTrail, which already records the IAM activity the scenario generates, and the GuardDuty to EventBridge to Lambda to SNS flow from Phase A stays a reference design for this account.

The setup also showed a practical point about the tooling. Lab instructions can lag behind the tools they describe, and checking the installed version and its help output is part of preparing an environment. Finally, the scenario includes a running EC2 instance, so it should be destroyed when the task is complete to avoid unnecessary cost and to leave the account clean.

Evidence Produced

The evidence for this phase is PhaseB_Resources.png. It shows the deployed CloudGoat resources (the Terraform resource list, the IAM roles and the scenario EC2 instance) together with the GuardDuty SubscriptionRequiredException. The secret access key shown in the scenario output was redacted.
