Phase B: AWS Security Fundamentals

Overview

This phase focused on applying basic AWS security controls and performing foundational cloud security tasks using the AWS Management Console and AWS CLI.

The activities covered identity and access management, centralized logging, S3 bucket enumeration, and secure EC2 deployment.

Objectives

- Verify AWS IAM users and apply the principle of least privilege.
- Enable CloudTrail logging across all AWS regions.
- List S3 buckets using the AWS CLI.
- Deploy an EC2 instance and apply basic security configurations.
- Document the security controls and evidence from each activity.

---

Task 1: IAM Users and Least Privilege

Activities

- Verified the existing IAM users in the AWS account.
- Created a test IAM user with minimal permissions.
- Used an administrator IAM account for tasks that required administrative privileges.
- Tested permissions using the AWS CLI.

Security Principle

The test user was configured with limited permissions to demonstrate the Principle of Least Privilege.

Users should only receive the permissions required to perform their assigned tasks.

Evidence

Screenshots documenting the IAM users and permissions are included in the project evidence.

---

Task 2: CloudTrail Logging

Activities

- Configured AWS CloudTrail.
- Enabled logging across all AWS regions.
- Verified that the trail was configured as a multi-region trail.
- Confirmed that CloudTrail logs were being delivered to an S3 bucket.

Security Principle

CloudTrail provides an audit trail of AWS API activity and helps with security monitoring, investigation, and accountability.

Evidence

Screenshots of the CloudTrail configuration and multi-region setting are included in the project evidence.

---

Task 3: S3 Bucket Enumeration Using AWS CLI

Activities

The AWS CLI was configured and used to interact with AWS services.

The following command was used to list S3 buckets:

aws s3 ls --region us-east-1 --endpoint-url https://s3.amazonaws.com

An explicit endpoint was used because of a connectivity issue with the default regional endpoint during the lab.

The command successfully returned the S3 bucket information.

Security Observation

S3 bucket enumeration can help administrators understand what storage resources exist within an AWS account.

Access to S3 resources should be controlled using appropriate IAM permissions.

Evidence

A screenshot of the successful AWS CLI S3 command and output is included in the project evidence.

---

Task 4: EC2 Instance Deployment

Configuration

Setting| Configuration
Operating System| Amazon Linux 2023
Instance Type| t3.micro
Storage| 8 GiB gp3
Public IP| Enabled
SSH| Port 22 / TCP
SSH Source| My IP
IAM Instance Profile| None
Metadata Version| IMDSv2 only
Metadata Hop Limit| 2
CPU Credit Specification| Standard
Purchase Option| None

Security Controls

Restricted SSH Access

SSH access was restricted to my IP address rather than allowing connections from anywhere on the internet.

IMDSv2

The EC2 instance was configured to require IMDSv2 for instance metadata access.

IAM Role

No IAM instance profile was attached because the task did not require the EC2 instance to access other AWS services.

CPU Credit Configuration

The instance was configured with Standard CPU credit behavior for this lab environment.

Verification

After deployment, the EC2 instance successfully passed 3/3 status checks.

Evidence

Screenshots documenting the running EC2 instance, instance details, and security group configuration are included in the project evidence.

---

Key Security Lessons

This phase reinforced several important cloud security concepts:

- Apply the principle of least privilege to IAM users.
- Enable centralized logging for visibility and investigation.
- Control access to S3 resources through IAM permissions.
- Restrict SSH access instead of exposing port 22 to the entire internet.
- Use IMDSv2 to strengthen EC2 metadata security.
- Document cloud configurations and security evidence.

Project Outcome

The AWS security fundamentals tasks were completed successfully.

The project provided practical experience with:

- AWS IAM
- AWS CloudTrail
- Amazon S3
- Amazon EC2
- AWS CLI
- Security Groups
- IAM permissions
- EC2 metadata security

This work forms part of my hands-on Cloud Security Engineer internship portfolio.
