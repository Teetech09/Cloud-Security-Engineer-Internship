Task 21: AWS CloudTrail Security Monitoring

Phase B: Setup Homelab / Target Environment

 Objective

Configure a new multi-Region AWS CloudTrail trail in a personal AWS Free Tier environment, with log storage in Amazon S3 and log file integrity validation enabled.

Environment

| Item | Value |
|---|---|
| Cloud provider | AWS (personal Free Tier account) |
| Service | AWS CloudTrail |
| Trail name | `SecurityTrail` |
| Trail scope | Multi-Region (all Regions) |
| Home Region | `us-east-1` (US East – N. Virginia) |
| S3 log bucket | `cloudtrail-securitylogs-titilayo` |
| Log file validation | Enabled |
| Event types | Management events (Read and Write) |

 Actions Taken

1. Signed in to the AWS Management Console and opened **CloudTrail**.
2. Selected Trails → Create trail and named the trail `SecurityTrail`.
3. Enabled multi-Region logging so that API activity from all Regions is recorded.
4. Created a new S3 bucket named `cloudtrail-securitylogs-titilayo` as the log storage location.
5. Enabled log file validation so that log file tampering can be detected.
6. Left data events and Insights events disabled to avoid additional cost, and kept management events (Read and Write) enabled.
7. Reviewed the configuration and created the trail.
8. Confirmed the trail status showed **Logging**.
9. Opened the S3 bucket and confirmed CloudTrail had created the `AWSLogs/` folder.
10. Verified that Region folders for `us-east-1`, `eu-west-1`, and `eu-north-1` were present, showing that the trail records activity from multiple Regions.

Note: SSE-KMS encryption and CloudWatch Logs integration were optional. -KMS and CloudWatch Logs were not enabled for this lab to avoid additional cost. Logs are encrypted at rest with the default SSE-S3 encryption."]*

Key Observations

- Trail status: `SecurityTrail` showed a status of Logging.
- Multi-Region coverage: The trail was configured as a multi-Region trail. The S3 bucket contained Region folders for `us-east-1`, `eu-west-1`, and `eu-north-1`.
- Bucket policy: When CloudTrail created the bucket, it attached a bucket policy that allows the CloudTrail service principal (`cloudtrail.amazonaws.com`) to deliver logs to the bucket.
- Log delivery: CloudTrail created the `AWSLogs/<account-ID>/CloudTrail/<Region>/` folder structure in the bucket, confirming that log delivery was working.
- Log file validation: Enabled, so digest files are generated to support integrity checks on delivered logs.

Security Relevance

A multi-Region trail gives visibility across the whole AWS account. Attackers often operate in Regions that an organisation does not normally use, and a single-Region trail would miss that activity. Storing logs in S3 keeps a record beyond the 90-day Event history, and log file validation allows analysts to detect whether log files have been modified or deleted. Together, these controls support:

- Detection of suspicious activity in unused Regions.
- Incident investigation with reliable historical records.
- Audit evidence and compliance monitoring.
- Protection of log integrity


  Completion Checklist

- [x] Trail created successfully.
- [x] Log file validation enabled.
- [x] S3 bucket policy allows delivery from CloudTrail.
- [x] Verified first log delivery.

Evidence Collected

Evidence 1: Trail configuration summary page

Screenshot of the `SecurityTrail` details page showing the trail name, multi-Region setting, logging status, log file validation, and S3 log location.

File location:

`evidence/PhaseB_CloudTrail_SecurityTrail_Config_Summary.png`

Evidence 2: S3 Region folders

Screenshot of the S3 bucket showing Region folders for `us-east-1`, `eu-west-1`, and `eu-north-1`, confirming multi-Region log delivery.

File location:

`evidence/PhaseB_S3_MultiRegion_Log_Folders.png`

> Account ID and other sensitive identifiers were blurred in the screenshots.

Cleanup and Cost Notes

The first copy of management events per Region is free. S3 storage and any optional services (KMS, CloudWatch Logs, data events) can add small charges. After the Week 6 tasks are complete, the trail can be stopped or deleted, and the S3 bucket emptied and deleted.

Conclusion

This phase set up a working multi-Region CloudTrail trail named `SecurityTrail`, with logs delivered to a dedicated S3 bucket and log file validation enabled. Seeing logs arrive from multiple Regions confirmed that the trail provides account-wide visibility. This environment will be used in the next phases to generate, search, and analyse CloudTrail events.

References

- [Creating a trail for your AWS account](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-a-trail-using-the-console-first-time.html)
- [Validating CloudTrail log file integrity](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html)
- [Amazon S3 bucket policy for CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-s3-bucket-policy-for-cloudtrail.html)
- [CloudTrail pricing](https://aws.amazon.com/cloudtrail/pricing/)
