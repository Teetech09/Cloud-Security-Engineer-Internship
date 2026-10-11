Phase C — Identity-Based Access Operations

Objective
Test least-privilege enforcement by performing allowed and denied AWS operations through named CLI profiles and Boto3.

Initial Observation
The `dev-user` initially had the AWS-managed `ReadOnlyAccess` policy. Both S3 bucket listing and EC2 instance description succeeded because the policy permitted both read operations.

The task expected EC2 access to be denied. To meet this intended restriction, I replaced `ReadOnlyAccess` with a custom policy allowing only S3 bucket listing.

Permission Hardening
In the AWS console, I opened:

IAM → Users → dev-user → Permissions → Add permissions → Create inline policy**

I created `DevBucketListingOnly` using:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowBucketListing",
      "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets",
      "Resource": "*"
    }
  ]
}
```

I then removed `ReadOnlyAccess` from `dev-user`. The `ops-user` retained its lab-required `AdministratorAccess` policy.

The wildcard resource is required for this account-level bucket-listing action. The policy does not grant access to objects inside the buckets.

CLI Access Tests

dev-user
```powershell
aws s3 ls --profile dev-user --region us-east-1
aws ec2 describe-instances --profile dev-user --region us-east-1 --no-cli-pager
```

ops-user
```powershell
aws s3 ls --profile ops-user --region us-east-1
aws ec2 describe-instances --profile ops-user --region us-east-1 --no-cli-pager
```

Results

| Identity | Operation | Result |
|---|---|---|
| dev-user | List S3 buckets | Successful |
| dev-user | Describe EC2 instances | UnauthorizedOperation |
| ops-user | List S3 buckets | Successful |
| ops-user | Describe EC2 instances | Successful |
| dev-user through Boto3 | List S3 buckets | Successful; three buckets returned |

The EC2 denial stated that no identity-based policy allowed `ec2:DescribeInstances`. This demonstrated the intended restriction for that action.

Boto3 Test
Python was initially unavailable, so I installed Python and then installed Boto3:

```powershell
python -m pip install boto3
python -c "import boto3; print(boto3.__version__)"
```

I executed the following command using the existing named profile:

```powershell
python -c "import boto3; session = boto3.Session(profile_name='dev-user', region_name='us-east-1'); s3 = session.client('s3'); response = s3.list_buckets(); print('S3 bucket listing succeeded'); print('\n'.join(bucket['Name'] for bucket in response['Buckets']))"
```

The request succeeded and returned three bucket names. No credentials were embedded in the code.

Troubleshooting
An S3 request initially returned a connection error for the regional endpoint. The request succeeded on retry. I treated this as a connectivity issue rather than evidence of a permission denial.

The initial successful EC2 request under `dev-user` was consistent with ReadOnlyAccess. Narrowing the policy produced the intended denial.

Security Analysis
The tests demonstrated that permissions attached to an identity determine which API actions it can perform. The narrowed development policy allowed bucket listing while denying EC2 instance description.

Successful bucket listing does not demonstrate access to bucket contents. Likewise, these tests validate the specific actions tested, rather than every permission held by either user.

Evidence
- [CLI successes and EC2 denial](./phaseC_access_tests.png)
- [Boto3 bucket-listing result](./phaseC_boto3_list.png)

Account identifiers and sensitive resource information are redacted from public screenshots. Credentials are excluded from this repository.

Lessons Learned
- Read-only access can span multiple AWS services.
- Least privilege requires permissions tailored to the intended task.
- Authentication checks and permission tests serve different purposes.
- Boto3 can reuse named CLI profiles without hardcoding credentials.
- Connectivity errors must be distinguished from authorisation denials.

 Completion Checklist
- [x] Verified successful S3 listing for dev-user.
- [x] Verified denied EC2 access for dev-user.
- [x] Verified successful S3 and EC2 requests for ops-user.
- [x] Listed S3 buckets programmatically using Boto3.
- [x] Uploaded redacted CLI and Boto3 evidence.

Status
Execution complete — evidence upload pending.
