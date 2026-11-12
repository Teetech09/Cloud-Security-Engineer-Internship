Phase-E CybrLabs – AWS Account and Organization ID Discovery

Objectives

Completed two AWS security lab exercises to explore how S3 bucket access and IAM permissions can reveal AWS account and organization identifiers.

The exercises were:

- Discover AWS Account ID via S3 Bucket
- Discover AWS Organization ID via S3 Bucket

Tools and Technologies

- Amazon Web Services (AWS)
- AWS CLI
- Python
- `s3-account-search`
- `conditional-love`
- IAM and Amazon S3

Actions Taken

- Configured and verified AWS lab credentials.
- Enumerated IAM roles and examined their permissions.
- Identified the target S3 bucket and relevant access policies.
- Used `s3-account-search` to discover the AWS account ID through S3 bucket access.
- Used `conditional-love` to progressively discover the AWS Organization ID through IAM policy condition evaluation.
- Reviewed the security implications of IAM permissions, resource ownership and cross-account access.

Key Findings

Successfully recovered the AWS account ID and AWS Organization ID through the two lab exercises.

The exercises demonstrated how overly permissive access and IAM policy condition evaluation can expose useful organizational information during cloud security reconnaissance.

Evidence

- PhaseE_CybrLab1_completion.png – AWS Account ID discovery completion.
- PhaseE_CybrLab2_completion.png – AWS Organization ID discovery completion.

Lessons Learned

Gained practical experience with AWS IAM role enumeration, S3 permissions, cross-account access, AWS identity discovery and IAM condition keys. Also improved my understanding of how cloud misconfigurations and permission evaluation can create information-disclosure risks.

AI Tool Usage Declaration

ChatGPT was used for PowerShell command guidance, troubleshooting, understanding AWS CLI outputs and technical documentation. All lab activities were performed in the CybrLabs environment.
