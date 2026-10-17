Phase B — Environment Setup

Objective

Deploy the IAM Vulnerable environment in a dedicated AWS sandbox to study misconfigured IAM permissions and privilege escalation paths.

 Updated Deployment Method

The original instructions described Docker containers and a local web interface on port 8080.

My Mentor provided the BishopFox IAM Vulnerable repository and confirmed that this environment uses Terraform to deploy AWS resources. I followed the updated deployment method.

Repository: https://github.com/BishopFox/iam-vulnerable

Actions Taken

1. Installed and verified Terraform and Git on Windows.
2. Cloned the IAM Vulnerable repository.
3. Reviewed `main.tf` and `variables.tf`.
4. Corrected formatting in the enabled module blocks.
5. Kept `privesc-paths` and `tool-testing` enabled.
6. Left optional service modules disabled.
7. Ran `terraform init` and `terraform validate`.
8. Generated and reviewed the deployment plan.
9. Configured Terraform to use my dedicated sandbox profile.
10. Applied the saved sandbox plan and verified the deployed IAM roles.

AWS Profile Configuration

I configured a named AWS CLI profile, `personal-sandbox`, and explicitly selected it in `terraform.tfvars`:

```hcl
aws_local_profile = "personal-sandbox"
