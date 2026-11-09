Task 21: AWS CloudTrail Security Monitoring

Phase A: Theory & Introduction

Objective

Understand the architecture of AWS CloudTrail and its role in cloud security monitoring, auditing, and incident investigation.

This phase focuses on understanding how AWS CloudTrail records API activities within an AWS account, the structure of CloudTrail event logs, and its integration with Amazon S3 and Amazon CloudWatch.

Actions Taken

The following activities were completed during this phase:

1. Reviewed AWS CloudTrail architecture and its role in monitoring activities within an AWS environment.
2. Studied how CloudTrail captures API calls initiated through the AWS Management Console, AWS CLI, SDKs, and other AWS services.
3. Examined the differences between management events and data events, including their security monitoring purposes.
4. Reviewed the configuration differences between single-Region and multi-Region trails.
5. Studied how CloudTrail integrates with Amazon S3 for log storage and Amazon CloudWatch for monitoring and alerting.
6. Accessed AWS CloudTrail through the AWS Management Console and navigated to **Event History**.
7. Examined an actual `CreateUser` event generated within the AWS lab environment.
8. Inspected the event's JSON record to identify important fields, including the user identity, event timestamp, event source, API operation, AWS Region, and source IP address.
9. Captured a screenshot of the CloudTrail event JSON as practical evidence.

Key Observations

During the practical examination of the CloudTrail event record, the following observations were made:

- Event Identity:The `userIdentity` field identified the activity as being initiated by an IAM user named `cloudgoat-lab`.
- Event Source: The `eventSource` field showed `iam.amazonaws.com`, indicating that the activity involved AWS Identity and Access Management.
- Event Name: The `eventName` field displayed `CreateUser`, indicating an IAM user creation operation.
- Event Timestamp:The `eventTime` field recorded the activity as `2026-11-07T12:19:42Z`.
- AWS Region: The `awsRegion` field identified `us-east-1` (US East – N. Virginia) as the Region associated with the event.
- Source IP Address: The `sourceIPAddress` field provided information about the originating IP address of the API request.

The examination demonstrated how CloudTrail provides detailed information about activities performed within an AWS account.

 Security Relevance

CloudTrail plays an important role in AWS security monitoring by providing visibility into activities performed by users, roles, applications, and AWS services.

For example, unauthorised IAM user creation could introduce additional access paths into an AWS environment. By examining CloudTrail records, security analysts can investigate who initiated the operation, when it occurred, and where the request originated.

This information supports:

- Identity and access monitoring.
- Detection of suspicious account activities.
- Security incident investigation.
- Audit evidence collection.
- Security accountability and compliance monitoring.

 Evidence Collected

The following evidence was captured during the practical exercise:

Evidence 1: CloudTrail CreateUser Event JSON

Screenshot showing an actual `CreateUser` event record viewed through AWS CloudTrail Event History.

File location:

Evidence-PhaseA_CloudTrail_CreateUser_Event_JSON.png

Conclusion

This phase provided an understanding of AWS CloudTrail architecture and its importance in cloud security monitoring.

By reviewing the CloudTrail documentation and examining an actual IAM `CreateUser` event, I gained practical knowledge of how AWS records API activities and provides event details that can support security monitoring, auditing, and incident investigation.

The exercise reinforced the importance of maintaining visibility into IAM activities and monitoring changes within cloud environments.

  References

- [AWS CloudTrail Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
- [How CloudTrail Works](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-working-with-log-files.html)
- [CloudTrail Event Reference](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference.html)
- [CloudTrail Management and Data Events](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)
