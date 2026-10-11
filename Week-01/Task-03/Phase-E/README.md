Phase E — Hosted Labs Integration

Objective
Reinforce AWS CLI configuration, authentication, profile management, and automation skills through hosted labs.

Labs Completed

| Lab | Platform | Status |
|---|---|---|
| Getting Started with the AWS CLI | Cybr | Completed |
| AWS CLI Tips and Tricks | Cybr | Completed |
| Understand Authentication Mechanisms Using Boto3 | PwnedLabs | Optional — not completed |

Getting Started with the AWS CLI
I completed the hosted lab covering AWS CLI setup and authentication, including an introduction to IAM roles. This reinforced the profile configuration and identity concepts explored in earlier phases.

Evidence
[View lab completion](./phaseE_cli_setup_completion.png)

AWS CLI Tips and Tricks
I completed exercises covering:
- Named profiles and the `AWS_PROFILE` environment variable.
- Differences between `s3`, `s3api`, and `s3control`.
- Navigating AWS CLI documentation.
- Filtering API responses with `--query` and processing JSON with `jq`.
- Using scripts, aliases, and functions to improve efficiency.

 Selected Lab Commands
The following commands were used in the lab’s Bash environment:

```bash
# Find buckets whose names contain cli-tips
aws s3api list-buckets --query 'Buckets[?contains(Name, `cli-tips`)].Name'

# Store the matching bucket name in a variable
BUCKET=$(aws s3api list-buckets --query 'Buckets[?contains(Name, `cli-tips`)].Name' --output text)

# Display the selected bucket
echo "Working with bucket: $BUCKET"

# Inspect the bucket's public access block configuration
aws s3api get-public-access-block --bucket "$BUCKET"
```

The final command inspects a security setting; executing it alone does not establish that the bucket is secure.

Evidence
[View lab completion](./phaseE_cli_tips_completion.png)

Key Observations
Both labs displayed completion confirmations. The first reinforced CLI authentication and introduced roles, while the second developed practical techniques for finding commands, filtering results, and automating repetitive tasks.

The Bash syntax used in the hosted labs differs from PowerShell syntax used in my local environment.

