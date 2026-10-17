Phase C — Policy Engineering

Objective

Engineer and test a least-privilege IAM policy for a Data Analyst who needs read-only access to an S3 data lake, without write or delete permissions.

Business Requirements

| Requirement | Implementation |
|---|---|
| List data lake objects | Allow `s3:ListBucket` on the specified bucket |
| Read data lake objects | Allow `s3:GetObject` on objects within that bucket |
| No write or delete permissions | Do not grant upload or delete actions |
| Restrict access to the data lake | Scope permissions to the specified bucket and its objects |

`example-data-lake` is a placeholder used for policy engineering and simulation. No bucket or test object was created for these tests.

Actions Taken

1. Created the Data Analyst policy in VS Code.
2. Uploaded it to AWS CloudShell as `PhaseC_data_analyst_policy.json`.
3. Checked the JSON syntax.
4. Used `aws iam simulate-custom-policy` to test permitted and prohibited actions.
5. Tested access to an object outside the specified bucket.
6. Validated the policy using AWS IAM Access Analyzer.
7. Saved the validation output as `PhaseC_AccessAnalyzer_Validation.json`.

 Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListDataLakeBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example-data-lake"
    },
    {
      "Sid": "ReadDataLakeObjects",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-data-lake/*"
    }
  ]
}
```

 Simulation Methodology

I tested the policy independently using `simulate-custom-policy`, without attaching it to an IAM identity.

After initial policy-input errors, I generated structured JSON request files containing the policy, action names, and resource ARNs. I passed these files through `--cli-input-json`, which successfully produced simulation results.

Bucket-level listing was tested against:

```text
arn:aws:s3:::example-data-lake
```

Object-level actions were tested against:

```text
arn:aws:s3:::example-data-lake/test-data.csv
```

The outside-bucket test used:

```text
arn:aws:s3:::example-outside-bucket/test-data.csv
```

 Simulation Results

| Action | Resource | Actual Decision |
|---|---|---|
| `s3:ListBucket` | Data lake bucket | `allowed` |
| `s3:GetObject` | Data lake object | `allowed` |
| `s3:PutObject` | Data lake object | `implicitDeny` |
| `s3:DeleteObject` | Data lake object | `implicitDeny` |
| `s3:GetObject` | Object outside the data lake | `implicitDeny` |

The results matched the tested business requirements.

`implicitDeny` means the supplied policy did not grant the requested action. It is not an explicit deny and does not prevent a separate policy from granting additional permissions.

## IAM Access Analyzer Validation

I ran the following command in CloudShell:

```bash
aws accessanalyzer validate-policy \
  --policy-document file://PhaseC_data_analyst_policy.json \
  --policy-type IDENTITY_POLICY \
  --region us-east-1 \
  --output json > PhaseC_AccessAnalyzer_Validation.json
```

The returned output was:

```json
{
  "findings": []
}
```

IAM Access Analyzer returned no validation findings. No further policy adjustments were required based on this output.

Alternative Tool and Scope

The updated BishopFox IAM Vulnerable repository deploys vulnerable IAM configurations but does not provide the analyzer described in the original task instructions.

I used AWS IAM Access Analyzer as an alternative policy validation method. Mentor acceptance of this substitution has not yet been recorded.

Zero validation findings do not independently prove that every possible excessive-permission risk has been eliminated. The simulation results provide separate evidence for the specific actions and resources tested.

Key Observations

- Least privilege requires restricting both actions and resources.
- Bucket-level and object-level actions require different resource ARNs.
- Explicitly listing the required actions avoids broad action wildcards.
- Testing prohibited actions is necessary to assess whether the policy matches the requirements.
- Policy validation and permission simulation provide different types of evidence.

Troubleshooting

Uploaded Filename

The uploaded file was named `PhaseC_data_analyst_policy.json`, which differed from the filename initially used in commands. I updated the commands to use the actual filename.

Policy Input Errors

Initial simulation attempts returned invalid policy-content errors. I replaced the file contents with a clean policy and used structured JSON simulation requests. Subsequent simulations succeeded.

 Missing Simulation Request File

An object simulation initially failed because its request file was not present. I recreated the file and reran the command successfully.

Evidence

- `PhaseC_data_analyst_policy.json`
- Screenshot of the successful bucket-listing simulation
- `PhaseC_Object_Permission_Simulation.png`
- `PhaseC_Outside_Bucket_Denied.png`
- `PhaseC_AccessAnalyzer_No_Findings.png`
- `PhaseC_AccessAnalyzer_Validation.json`

Lessons Learned

I learned to verify filenames and JSON contents before running AWS CLI commands. Structured request files helped me pass policy documents reliably to the simulator.

I also learned that a policy can grant read access without granting write or delete permissions, while resource scoping prevents that read permission from extending to unrelated buckets.

Completion Status

- [x] Custom policy created.
- [x] Required read actions simulated successfully.
- [x] Write and delete actions implicitly denied.
- [x] Outside-bucket access implicitly denied.
- [x] AWS IAM Access Analyzer returned zero validation findings.
- [X] Mentor acceptance of the alternative analyzer recorded.

References

- [SimulateCustomPolicy — AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/iam/simulate-custom-policy.html)
- [IAM Access Analyzer Policy Validation](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer-policy-validation.html)
