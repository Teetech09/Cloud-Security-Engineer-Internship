Phase B: Set Up Homelab Environment

 Objective

The objective of this phase was to set up a local homelab in Docker that simulates exploitable IAM configurations and to confirm that the AWS CLI can be pointed at it for testing.

Actions Taken

I started by checking the setup instructions for this phase, which refer to an iam-vulnerable Docker environment with a web interface on port 8080. The BishopFox iam-vulnerable project is deployed with Terraform into a real AWS account and has no Docker image or dashboard, so I used a different local simulator to meet the objective: Moto, an open-source AWS emulator that supports IAM and runs as a container. I did not use LocalStack because its image now requires an account and an auth token.

Docker Desktop was already installed on my Windows laptop, but my WSL Ubuntu distribution could not find the `docker` command. I enabled WSL integration for Ubuntu in the Docker Desktop settings, reopened Ubuntu, and confirmed that `docker --version` returned version 29.8.2. I then started the simulator with `docker run -d --name iam-sim -p 8080:5000 motoserver/moto:latest`, which maps port 8080 on my machine to Moto's port 5000. I confirmed the container was running with `docker ps`, and I checked that the service responded by requesting `http://localhost:8080/moto-api/` with `curl`, which returned the Moto status page.

Next I configured a separate AWS CLI profile named `iam-local` with the placeholder credentials `test` and `test` in `us-east-1`, so my real AWS profiles stayed untouched. I pointed the CLI at the simulator with `--endpoint-url http://localhost:8080` and listed the users to confirm the connection. To recreate a vulnerable configuration, I created a user called `privesc-user` and a policy called `RiskyDevPolicy` that allows `iam:PassRole`, `iam:AttachUserPolicy`, `iam:PutRolePolicy` and `iam:CreatePolicyVersion` on all resources, then attached the policy to the user and confirmed the attachment with `list-attached-user-policies`.

Key Observations

The Moto container started cleanly and was reachable on port 8080, and the AWS CLI worked against it just as it would against AWS, apart from the extra `--endpoint-url` flag. Everything in the simulator is local, so I could create deliberately dangerous IAM resources with no risk to a real account and no cost.

This setup has a limit that I want to record. Moto does not enforce IAM permissions by default, so I could create the vulnerable configuration and inspect it, but I could not actually exploit it, because the simulator will accept any action regardless of the policies. Real privilege escalation testing still needs a real AWS account, like the iam-vulnerable deployment I ran earlier with Terraform. The simulator is useful for learning how these resources are structured and for testing tooling against them, and it fits the sample policy I analysed with Cloudsplaining in Phase A.

Evidence Produced

PhaseB_docker.png is a screenshot of the `docker ps` output showing the `iam-sim` container running with port 8080 mapped to the simulator. PhaseB_cli.png` is a screenshot of the AWS CLI output showing `RiskyDevPolicy` attached to `privesc-user` through the `iam-local` profile.
