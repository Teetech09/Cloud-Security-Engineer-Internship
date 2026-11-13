Phase B: Setup Homelab Environment

Objective
The objective of this phase was to deploy the BishopFox iam-vulnerable environment with Terraform in a dedicated AWS lab account, and to confirm that the AWS CLI was connected to that account. iam-vulnerable creates deliberately misconfigured IAM users, roles, and policies, which later phases use to practise enumerating weak credentials and privilege-escalation paths.

Actions Taken
I worked in Ubuntu under WSL2 on a Windows 10 laptop. Before starting, I confirmed that Git, Terraform (v1.17.0), and the AWS CLI (v2.38.4) were installed. My first attempt opened WSL's bare system shell, where none of these tools existed, and the startup output reported that the Ubuntu disk had been mounted read-only. I exported a backup of the distribution with `wsl --export`, ran `wsl --shutdown`, and reopened Ubuntu directly with `wsl -d Ubuntu`. A write test in the home directory then succeeded, so I continued from there.

I verified my AWS identity with `aws sts get-caller-identity` before deploying, to make sure I was working in the lab account. I then cloned the repository with `git clone https://github.com/BishopFox/iam-vulnerable` and changed into the directory. The Terraform files sit in the repository root, and the provider settings are controlled by optional variables in `terraform.tfvars.example`, all commented out. I did not create a `terraform.tfvars` file, so Terraform used my default AWS credentials and the us-east-1 region.

I ran `terraform init` and `terraform plan`, which reported 265 resources to add, 0 to change, and 0 to destroy. I then ran `terraform apply`. Terraform only accepts a lowercase `yes` at the confirmation prompt, and my first attempt was typed into the shell instead, so I reran the apply and confirmed correctly. It finished with `Apply complete! Resources: 265 added, 0 changed, 0 destroyed.` Finally, I ran `aws sts get-caller-identity` again to confirm that the CLI connection was active.

Key Observations
The plan showed only new resources, so the deployment did not modify anything that already existed in the account. The environment is made up of many IAM users, roles, and policies, each built around a specific privilege-escalation path, so it is intentionally insecure and belongs only in a dedicated lab account. Using the default credentials meant that the resources were created in whichever account the CLI was authenticated against, which is why I checked the caller identity before running the apply. The WSL disk issue was a useful reminder to take a backup before troubleshooting, and to check the shell prompt and working directory before running lab commands. I left the environment deployed because Phase C depends on it, and it will be removed with `terraform destroy` once the lab work is complete.

Evidence Produced
The screenshot `phaseB_setup.png` shows the terminal with the successful Terraform apply message and the `aws sts get-caller-identity` output confirming the CLI connection. The account ID has been blurred.
