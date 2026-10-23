Phase C: Core Execution: Enumeration & Policy Analysis

Objective
Enumerate all accessible buckets and analyze their ACLs and policies for exposure risks.

Actions Taken (Phase C)

I redeployed the CloudGoat scenario `cloud_breach_s3`, since I had destroyed the Phase B lab. The new bucket is `cg-cardholder-data-bucket-cgiduv0zua8xhw`. I then enumerated every bucket in the account and analyzed its ACL and policy.

1. Enumerated buckets (8 found):
```bash
export AWS_PROFILE=cloudgoat
aws s3api list-buckets --query "Buckets[].Name"
```

2. Retrieved the ACL and policy for every bucket:**
```bash
for b in $(aws s3api list-buckets --query "Buckets[].Name" --output text); do
  aws s3api get-bucket-acl --bucket "$b" > "acl_$b.json"
  aws s3api get-bucket-policy --bucket "$b" > "policy_$b.json" 2> "policy_$b.err"
done
```
Three buckets returned `NoSuchBucketPolicy`, meaning they have no bucket policy.

3. Extracted principals with jq. `get-bucket-policy` returns the policy as an escaped string, so it has to be decoded first:
```bash
jq -r '.Policy | fromjson | .Statement[].Principal' policy_<bucket_name>.json
```

4. Checked ACLs for public grantees:
```bash
jq -r '.Grants[] | select(.Grantee.URI != null) | .Grantee.URI' acl_<bucket_name>.json
```

5. Queried Block Public Access settings with Steampipe:**
```bash
steampipe query "select name, block_public_acls, block_public_policy, bucket_policy_is_public from aws_s3_bucket;"
```

Key Observation (Phase C)

Findings:
- ACLs: no bucket grants access to `AllUsers` or `AuthenticatedUsers`.
- Policies: 5 buckets have a policy and 3 have none. The principals are AWS services (CloudTrail, Config, S3 logging) and one specific IAM user (`ops-user`). One policy, on `student-s3-lab-teetech-20261020`, uses `"Principal": "*"`.
- Steampipe: all 8 buckets have `block_public_acls = true`, `block_public_policy = true` and `bucket_policy_is_public = false`.

The wildcard principal is not a public bucket.The policy with `"Principal": "*"` is a hardening control:
```json
{
  "Sid": "DenyAnonymousAccess",
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Condition": { "Null": { "aws:PrincipalArn": "true" } }
}
```
It denies all S3 actions on that bucket for requests with no principal ARN, meaning anonymous requests. A wildcard principal has to be read together with `Effect`, `Action` and `Condition`. With `Allow` and no condition it would be dangerous, but with `Deny` and an anonymous-request condition it protects the bucket.

No publicly accessible buckets were found. That includes the CloudGoat bucket `cg-cardholder-data-bucket-cgiduv0zua8xhw`, which is private by design, because this scenario is about reaching the data through stolen credentials rather than public access. The bucket `student-s3-public-sim-teetech-20261020` has "public" in its name but has no public ACL, no policy and Block Public Access enabled, so the name is misleading and shouldn't be trusted as evidence.

Defence in depth: `student-s3-lab-teetech-20261020` is protected by two independent controls, Block Public Access and the anonymous-deny policy.

Tooling notes:
- The `jq` command in the task needed `fromjson`, because the policy is returned as an escaped string.
- Steampipe's AWS plugin first failed to install because WSL's DNS couldn't resolve `ghcr.io`. Setting explicit DNS servers fixed it.

Evidence Produced

- phaseC_public_buckets.png`: Steampipe output listing all 8 buckets and their Block Public Access and public-policy status.
- phaseC_principal_analysis.png`: jq output extracting the principals from each bucket policy.

[Public bucket check](phaseC_public_buckets.png)

Completion Checklist
- [x] All bucket ACLs retrieved
- [x] Policies analyzed with jq
- [x] Publicly accessible buckets identified via Steampipe (none found)

## Cleanup
- [ ] Ran `cloudgoat destroy cloud_breach_s3`
- [ ] Deleted the lab IAM user's access key
