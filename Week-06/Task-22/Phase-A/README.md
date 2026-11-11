 Phase A: Theory & Introduction

Objective

Understand how IAM and S3 activities are recorded in AWS CloudTrail and how attackers may exploit IAM and S3 misconfigurations.

Actions Taken

- Reviewed the structure of AWS CloudTrail event JSON records.
- Examined key event fields, including `eventName`, `userIdentity`, `sourceIPAddress`, `requestParameters`, `eventTime`, and `eventSource`.
- Reviewed three IAM event examples: `CreateUser`, `AttachUserPolicy`, and `CreateAccessKey`.
- Analysed how legitimate administrative activities can differ from suspicious actions, particularly unexpected privilege escalation through the attachment of the `AdministratorAccess` policy.
- Reviewed S3 object-level activities, including `GetObject`, `PutObject`, `DeleteObject`, and `ListBucket`.

 Key Observations

CloudTrail provides visibility into IAM and S3 activities, making it useful for identifying suspicious account management, permission changes, and potential data exposure.

Unexpected attachment of administrative policies, unauthorised access key creation, and unusual S3 object access may indicate privilege escalation, persistence, reconnaissance, or possible data exfiltration.

The context of an event, including the identity performing the action, originating IP address, timestamp, and requested parameters, is essential when determining whether an activity is suspicious.

Security Findings

| Activity | Security Observation |
|---|---|
| CreateUser | Verify that account creation is authorised. |
| AttachUserPolicy | Unexpected AdministratorAccess attachment may indicate privilege escalation. |
| CreateAccessKey | Unauthorised key creation may indicate persistence or credential compromise. |
| GetObject | Unusual object downloads may indicate possible data exfiltration. |
| ListBucket | Unexpected bucket enumeration may indicate reconnaissance. |

Evidence

PhaseA_evidence.txt – Annotated CloudTrail JSON event highlighting suspicious IAM activity and relevant security fields.

Conclusion

This phase established an understanding of how AWS CloudTrail records IAM and S3 activities and how event analysis can help identify suspicious behaviour. Particular attention was given to excessive permissions, unauthorised credential creation, and unusual S3 access patterns.
