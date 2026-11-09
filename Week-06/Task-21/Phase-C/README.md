Task 21: AWS CloudTrail Security Monitoring

Phase C: Core Execution – Event Analysis

Objective

Analyse captured AWS CloudTrail events to identify potential security incidents, using activity generated in the lab environment and an AI-assisted summary of the event records.

Environment

| Item | Value |
|---|---|
| Trail | `SecurityTrail` (multi-Region, log file validation enabled) |
| Log bucket | `cloudtrail-securitylogs-titilayo` |
| Home Region | `us-east-1` (US East – N. Virginia) |
| Admin identity used for setup | `ops-user` (IAM user, AWS CLI in WSL Ubuntu) |
| Test identity | `cloudtrail-test-user` (temporary IAM user) |
| Test permission | `AmazonS3ReadOnlyAccess` (AWS managed policy) |

Actions Taken

I installed the AWS CLI in my WSL Ubuntu environment and configured it with an access key for the `ops-user` IAM user, then confirmed the connection with `aws sts get-caller-identity`. To generate security-relevant activity, I created a temporary IAM user named `cloudtrail-test-user`, attached the AWS managed policy `AmazonS3ReadOnlyAccess` to it, and created an access key for it. I configured the key as a separate CLI profile named `cloudtrail-test`, and confirmed that the profile authenticated successfully as the test user.

Using that profile, I deliberately attempted actions the user had no permission to perform: `aws iam list-users` and `aws ec2 describe-instances`. Both calls were rejected, with `AccessDenied` for IAM and `UnauthorizedOperation` for EC2. I then opened CloudTrail Event history, filtered the events by user name, and examined the `CreateUser`, `AttachUserPolicy`, `ListUsers`, and `DescribeInstances` records. I also exported Event history to CSV to confirm the denied calls had been recorded. I searched Event history for activity by the root user to check for root account usage. I downloaded a CloudTrail log file from the S3 bucket, extracted it, and located the relevant events. I redacted account IDs, access key IDs, and IP addresses from the event records, and used an AI tool with the prompt *"Summarize this CloudTrail event in plain language and identify any suspicious actions"* to produce a plain-language summary of each event. Finally, I cleaned up by deleting the test user's access key, detaching the policy, deleting the test user, and clearing the test profile's credentials.

> Commands used (secrets omitted):
>
> ```bash
> aws iam create-user --user-name cloudtrail-test-user
> aws iam attach-user-policy --user-name cloudtrail-test-user \
>   --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
> aws iam create-access-key --user-name cloudtrail-test-user
> aws configure --profile cloudtrail-test
> aws iam list-users --profile cloudtrail-test
> aws ec2 describe-instances --region us-east-1 --profile cloudtrail-test
> ```

 Key Observations

CloudTrail recorded each of the actions above, including the failed ones. The denied calls were attributed to `cloudtrail-test-user` and carried an error code, which is what makes them detectable.

| Time (UTC, 2026-11-09) | Identity | Event | Result |
|---|---|---|---|
| Earlier in session | `ops-user` | `CreateUser` | Success |
| Earlier in session | `ops-user` | `AttachUserPolicy` | Success |
| 14:26:12 | `cloudtrail-test-user` | `ListUsers` | `AccessDenied` |
| 14:46:30 | `cloudtrail-test-user` | `GetCallerIdentity` | Success |
| 14:47:16 | `cloudtrail-test-user` | `ListUsers` | `AccessDenied` |
| 14:47:23 | `cloudtrail-test-user` | `DescribeInstances` | `Client.UnauthorizedOperation` |

Event type 1: Unauthorized API calls. The `ListUsers` event shows `errorCode: AccessDenied`, and the `errorMessage` states that no identity-based policy allows `iam:ListUsers`. The `DescribeInstances` event shows `Client.UnauthorizedOperation` for the same reason. The test user only had S3 read-only access, so IAM and EC2 calls were correctly blocked, and AWS recorded both attempts.

Event type 2: Policy changes. The `AttachUserPolicy` event shows `ops-user` attaching `arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess` to `cloudtrail-test-user`. The `requestParameters` field identifies the user and the policy ARN. This type of event is important to monitor, because a change to permissions can widen what an identity is able to do.

Event type 3: Root account usage.
- No root account activity was found when Event history was searched for the username `root`. This is the expected result for a well-managed account. Any root usage should be rare and investigated.*
- Root activity was found: `[event name]` at `[time]`, with `userIdentity.type` set to `Root`. 
Behaviour compared to Phase A. In Phase A, I examined an individual event record. In this phase, the same fields (`userIdentity`, `eventName`, `errorCode`, `sourceIPAddress`) made it possible to tell a successful administrative action apart from a failed attempt by a low-privileged identity.

Unexpected result. Some events did not appear in the console when I first searched for them, even though they were recorded. The CSV export of Event history confirmed that the `ListUsers` and `DescribeInstances` denials existed. The console search had been rejected because of an invalid filter value. The lesson is that search filters in Event history must be exact, and that the CLI (`aws cloudtrail lookup-events`) or an export can be used to confirm events.

Suspicious Patterns Documented

- Repeated denied calls from a new identity. `cloudtrail-test-user` made denied calls to two unrelated services (IAM and EC2) within about a minute, and a first denied attempt came about 20 minutes earlier. This resembles privilege probing, where someone with a stolen key checks what it can do.
- Create user, attach policy, create access key. This sequence in a short time can indicate persistence if it is not expected. In this lab, it was intentional.
- Permission changes by an administrator. Policy attachments should be reviewed against change approvals.
- Root usage. Should be rare and alerted on.

Detection ideas: A CloudWatch metric filter and alarm on `errorCode` values of `AccessDenied` or `UnauthorizedOperation`, an alert on IAM events such as `CreateUser`, `AttachUserPolicy`, and `CreateAccessKey`, and an alert on any `userIdentity.type` of `Root`.

AI-Assisted Summary

The AI tool was given a redacted copy of each event, with the account ID, access key ID, and source IP replaced by placeholders. Its summaries are saved in `event_analysis_titilayo.txt`.

In summary, the AI tool described the `ListUsers` and `DescribeInstances` events as failed attempts by a limited IAM user to access services it was not permitted to use, and flagged repeated denied calls as potentially suspicious. It described `AttachUserPolicy` as a permission change to an IAM user and noted that it should be verified as authorised. I reviewed these summaries against the raw event fields, and they matched what the records showed. *[Add any point where you disagreed with, or corrected, the AI output.]*

Completion Checklist

- [x] At least three distinct event types analysed.
- [x] AI-assisted summary created.
- [x] Suspicious patterns documented.

Evidence Produced



- evidence/PhaseC_CloudTrail_Unauthorized_API_Call_Annotated.png`: the denied `ListUsers` event, with `errorCode`, `errorMessage`, and `userIdentity` highlighted.
- evidence/PhaseC_CloudTrail_Policy_Change_Annotated.png`: the `AttachUserPolicy` event, with `policyArn` and `userIdentity` highlighted.
- evidence/PhaseC_CloudTrail_CreateUser_Annotated.png`: the `CreateUser` event.


> Account IDs, access key IDs, and IP addresses were redacted from all evidence.

Cleanup

The temporary test user, its access key, and its policy attachment were deleted, and the `cloudtrail-test` profile credentials were cleared from the CLI configuration. These cleanup actions generated additional `DeleteAccessKey`, `DetachUserPolicy`, and `DeleteUser` events in CloudTrail.

Conclusion

This phase showed how CloudTrail records both successful and failed API activity, and how the event fields can be used to identify unauthorised access attempts, permission changes, and root usage. Generating the activity myself and then finding it in the logs made the connection between an action and its audit record very clear. It also showed the value of alerting on denied calls and IAM changes in a real environment.

References

- [CloudTrail event reference](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference.html)
- [Viewing events with CloudTrail Event history](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)
- [Looking up events with the AWS CLI](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events-cli.html)
- [CloudWatch alarms for CloudTrail events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/monitor-cloudtrail-log-files-with-cloudwatch-logs.html)
