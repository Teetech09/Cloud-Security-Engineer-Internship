Phase E – Guided IAM Enumeration Labs

Objective

Practise using the AWS CLI to identify IAM identities, enumerate users’ permissions, retrieve policy documents, and interpret access to AWS resources.

Labs Undertaken

| Lab | Status |
|---|---|
| Cybr – Introduction to AWS IAM Enumeration | Completed |
| PwnedLabs – Intro to AWS IAM Enumeration | Enumeration and access exercises performed; flag retrieval blocked |

1. Cybr – Introduction to AWS IAM Enumeration

I completed the Cybr lab to strengthen my understanding of basic AWS CLI commands and IAM enumeration. The exercises helped me become more comfortable using the command line to inspect identity information and permissions.

This provided a foundation for the more detailed policy investigation in PwnedLabs.

## 2. PwnedLabs – Intro to AWS IAM Enumeration

### Actions Taken

I used AWS CLI in PowerShell with the credentials supplied by the lab.

- Confirmed the current identity using `aws sts get-caller-identity`.
- Retrieved details about the IAM user `dev01`.
- Checked group membership and confirmed that the user belonged to no IAM groups.
- Listed the managed policies attached directly to the user.
- Identified and retrieved the inline policy `S3_Access`.
- Listed versions of the customer-managed policy `dev01` and inspected its active version, `v8`.
- Enumerated the managed policy attached to the `BackendDev` role.
- Retrieved `BackendDevPolicy` and examined the role’s trust policy.
- Successfully assumed `BackendDev` and verified the role identity through a named CLI profile.
- Listed Secrets Manager metadata and retrieved the lab secret permitted by the role.
- Attempted to list the S3 bucket and download `flag.txt`.
- Investigated the quarantine policy responsible for the S3 access restriction.

Key Observations

| Observation | Interpretation |
|---|---|
| `dev01` had no group memberships. | No permissions were inherited through IAM groups. |
| The active `dev01` policy allowed `iam:Get*` and `iam:List*` on `*`. | The user could inspect IAM resources and policy documents. |
| `S3_Access` allowed `s3:ListBucket` and `s3:GetObject` for `hl-dev-artifacts`. | The inline policy granted bucket listing and object reading permissions. |
| The `BackendDev` trust policy explicitly named `dev01`. | The role trusted the lab user to assume it; role assumption succeeded. |
| `BackendDevPolicy` allowed EC2 inspection, secret listing, and reading one specified secret. | Assuming the role provided access to resources beyond the user’s directly demonstrated permissions. |
| `AWSCompromisedKeyQuarantineV3` denied `s3:ListBucket` and `s3:GetObject` without conditions. | The explicit deny overrode the inline S3 allow and blocked the flag retrieval path. |

### Troubleshooting

**DNS resolution failure**

An initial S3 request could not connect to the endpoint. PowerShell DNS checks returned incomplete resolution through the default resolver, while a query through `1.1.1.1` returned IP addresses. After troubleshooting and retrying, the request reached AWS and returned an authorization error.

**Region configuration**

A later download attempt used `eu-west-1`. I retried with `--region us-east-1` explicitly set. The request then returned `403 Forbidden`.

**S3 authorization restriction**

The bucket-listing error explicitly identified `AWSCompromisedKeyQuarantineV3` as the denying policy. I retrieved its active version, `v3`, and confirmed that it denied both S3 listing and object retrieval.

The object-download attempt returned `403 Forbidden`. Although that response alone did not identify the cause, the inspected quarantine policy independently confirmed a deny on `s3:GetObject`.

### Outcome and Limitation

I performed the IAM enumeration exercises, interpreted user and role policies, successfully assumed the BackendDev role, and retrieved the permitted lab secret.

I could not retrieve or submit the S3 flag. The lab user’s quarantine policy explicitly denied the required S3 actions. I documented this blocker rather than marking the PwnedLabs lab as fully completed.

 Evidence

- Cybr completion screenshot.
- PwnedLabs identity verification and user details.
- Empty group membership output.
- Attached managed policy list.
- `S3_Access` inline policy document.
- Active `dev01` policy version and document.
- `BackendDevPolicy` and role trust policy.
- Successful assumed-role identity verification.
- Secrets Manager metadata with sensitive values excluded.
- S3 access-denied output.
- Quarantine policy showing the denied S3 actions.

Lessons Learned

- Listing policy names is different from retrieving and interpreting policy documents.
- The default managed policy version determines the policy currently in effect.
- Permissions policies define what an identity can do, while role trust policies define who can assume a role.
- Reading a role’s permissions does not automatically grant those permissions to the current user.
- An explicit deny overrides an allow.
- Connection failures and authorization failures require different troubleshooting approaches.
- Successful enumeration does not mean every resource is accessible or that the flag has been retrieved.

Portfolio Data Handling

Access keys, secret keys, session tokens, database passwords, and flag values are excluded from the public evidence.
