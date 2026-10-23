Week 3, Task 11 – Phase A: Theory & Introduction

Objective
Understand how S3 exposure occurs through misconfigurations such as public ACLs, overly permissive bucket policies, or disabled Block Public Access settings.

Actions Taken (Phase A)

I reviewed the structure of an S3 bucket policy JSON file using a deliberately misconfigured sample:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-company-data/*"
    }
  ]
}
```

I identified the key fields: `Effect` (Allow or Deny), `Principal` (who the rule applies to), `Action` (what they can do), `Resource` (which bucket or objects) and the optional `Condition`. I then used an AI assistant to interpret the policy and explain what `"Principal": "*"` implies (screenshot below). Finally, I compared ACLs with bucket policies and read about how attackers find open buckets.

Key Observation (Phase A)

- **`"Principal": "*"` means anyone**, including anonymous internet users with no AWS account. With `Effect: Allow` and `s3:GetObject` and no `Condition`, anyone who knows or guesses an object's URL can download it.
- **ACLs vs policies:** ACLs are a legacy mechanism with coarse permissions (READ, WRITE, FULL_CONTROL) and no conditions. Bucket policies are JSON documents with fine-grained actions, resources and conditions (IP, VPC, MFA, org). Either can expose data, and AWS recommends disabling ACLs.
- **Block Public Access is the safety net.** It overrides public ACLs and public policies. A bucket is typically exposed when Block Public Access is disabled **and** a public ACL or policy is added.
- **How attackers find open buckets:** they guess predictable names (company name plus words like `backup`, `dev` or `logs`), read the response to tell whether a bucket exists and whether it's open, use enumeration tools, and search leaked names in code, DNS and documentation.
- **Defences:** enable Block Public Access, disable ACLs, use IAM Access Analyzer for S3, add AWS Config rules, use Macie for sensitive data, and keep sensitive information out of bucket names.

Evidence Produced

- `phaseA_policy_analysis.doc`: screenshot of the AI assistant's explanation of the sample bucket policy and what `"Principal": "*"` implies.

[Policy analysis](phaseA_policy_analysis.png)

Completion Checklist
- [x] Reviewed sample bucket policy JSON
- [x] Understood ACL vs policy differences
- [x] Used AI assistant to interpret a sample policy
