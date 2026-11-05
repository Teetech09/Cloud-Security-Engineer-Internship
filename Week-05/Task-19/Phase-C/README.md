Phase C: Core Execution & Security Analysis

 Objective
The objective of this phase was to analyze how permissions propagate when an S3 event invokes a Lambda function, and to identify the impact of an over-permissive execution role by observing the behavior in CloudWatch Logs and CloudTrail.

Actions Taken
To make permission changes visible, I updated `SecureLambdaDemo` so that for each S3 event it logs the object key and then attempts three actions using its execution role: reading the triggering object (`s3:GetObject`), listing all buckets in the account (`s3:ListAllMyBuckets`), and deleting the triggering object (`s3:DeleteObject`). Each attempt is logged as `ALLOWED` or `DENIED`. The function only logs counts and status codes, never object contents or bucket names, and it never writes to the bucket, so it cannot re-trigger itself.

I ran the test in three stages:

1. Baseline (least privilege). I uploaded `before.txt` to trigger the function while `LambdaExecRole` had only `AWSLambdaBasicExecutionRole` and the inline `S3ReadOnlyLabBucket` policy.
2. Over-permission test. I temporarily attached an inline policy named `TempOverPermissive` allowing `s3:*` on `*`, waited for it to propagate, and uploaded `after.txt`.
3. Revert. I deleted `TempOverPermissive`, confirmed with `list-role-policies` and `list-attached-role-policies` that only the original two policies remained, and uploaded `revert.txt`.

I then used CloudTrail to look up the `ListBuckets` events and the IAM policy change events.

Key Observations
The invocation itself was identical in all three stages. S3 invoked the function under the same resource-based policy each time, and Lambda ran it with temporary credentials from the execution role. Only the reach of the execution role changed, and the function acted on whatever the role allowed.

| Stage | GetObject | ListAllMyBuckets | DeleteObject |
|---|---|---|---|
| Baseline (least privilege) | Allowed | Denied (AccessDenied) | Denied (AccessDenied) |
| Over-permissioned (`s3:*` on `*`) | Allowed | Allowed (9 buckets visible) | Allowed (HTTP 204) |
| After revert | Allowed | Denied (AccessDenied) | Denied (AccessDenied) |

In the over-permission stage, `after.txt` was deleted by the function itself, which shows the real impact of the misconfiguration: a function that only needs to read one bucket could enumerate every bucket in the account and destroy data.

CloudTrail confirmed the same pattern. The `ListBuckets` events were recorded under `assumed-role/LambdaExecRole/SecureLambdaDemo`, showing the function acting through temporary credentials. They show `AccessDenied` at the baseline and after the revert, and `Success` during the over-permission window, matching the CloudWatch logs to the second. CloudTrail also recorded the `PutRolePolicy` and `DeleteRolePolicy` calls that added and removed the temporary policy. S3 object-level actions such as `GetObject` and `DeleteObject` do not appear in CloudTrail Event history because they are data events, which are not logged unless a trail with data events enabled is configured.

Misconfigurations and Mitigations
The misconfiguration tested here is a wildcard policy (`s3:*` on `*`) on a function execution role. It is dangerous because Lambda functions process events that can contain attacker-influenced data, so any code flaw in the function inherits everything the role allows.

Mitigations identified during this phase:
- Apply least privilege to every execution role, naming specific actions and specific resource ARNs rather than wildcards.
- Scope the resource-based invoke permission with `SourceArn` and `SourceAccount` so only the intended bucket can trigger the function.
- Use IAM Access Analyzer to review policies and flag overly broad access.
- Alert on CloudTrail `PutRolePolicy`, `AttachRolePolicy,`, and similar IAM changes to execution roles.
- Enable CloudTrail data events for sensitive buckets if object-level auditing is required.
- Keep temporary permission changes short-lived and verify the revert, as done here.

Evidence Produced
* PhaseC_logs.png: CloudWatch Logs output showing the `PROBE` results for the baseline, over-permission and post-revert runs in time order.
* PhaseC_cloudtrail.png: CloudTrail `ListBuckets` events for `LambdaExecRole/SecureLambdaDemo`, showing `AccessDenied`, `Success` and `AccessDenied`.
* PhaseC_cloudtrail_iam.png: CloudTrail `PutRolePolicy` and `DeleteRolePolicy` events showing the temporary policy being added and removed (include only if you captured it).

![CloudWatch logs before and after modification](phaseC_logs.png)

![CloudTrail ListBuckets events](phaseC_cloudtrail.png)
