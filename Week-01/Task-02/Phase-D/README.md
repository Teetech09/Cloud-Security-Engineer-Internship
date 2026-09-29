# Phase D — Safe Vulnerable Lab Deployment

## Objective
Review CloudGoat and iam-vulnerable for deployment in private subnets without public IPs or externally exposed security groups.

## Configuration Review
I cloned both repositories and inspected the CloudGoat `ec2_ssrf` Terraform files before deployment. The scenario defines a public subnet with a route to an internet gateway. Its EC2 configuration uses that subnet, assigns a public IP, and uses the public IP for SSH file provisioning.

The current `iam-vulnerable` repository deploys primarily account-wide IAM resources through Terraform. IAM users, roles, and policies cannot be placed in subnets or verified with `aws ec2 describe-instances`.

## Decision and Status
I did not deploy either lab. The CloudGoat configuration conflicted with the private-subnet and no-public-IP requirements, and the stated subnet checks do not apply to the IAM resources in iam-vulnerable. Afzar confirmed that I should document this conflict and continue other work without deploying a lab that violates the safety requirements.

## Completion Checklist
- [ ] Labs deployed successfully — not attempted due to the configuration conflict
- [ ] No public IPs assigned — no lab EC2 instances were deployed
- [ ] Security groups restricted to internal traffic — no lab security groups were deployed
- [ ] Isolation verified using `aws ec2 describe-instances` — deployment-based verification not applicable

## Evidence
- Excerpts or screenshots from `ec2_ssrf/terraform/vpc.tf` showing the internet-gateway route
- Excerpts or screenshots from `ec2_ssrf/terraform/ec2.tf` showing the public subnet, public IP, and SSH connection to `self.public_ip`
- The iam-vulnerable repository documentation describing its IAM resources and Terraform deployment
- Afzar’s reply confirming the decision not to deploy
