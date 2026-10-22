 Phase D: Object-Level Permissions and Logging

Objective

Evaluate object ACL behavior, enable S3 server access logging and verify that read/write activity is recorded.

Environment

| Component | Configuration |
|---|---|
| Source bucket | secure-s3-lab-teetech-20261021 |
| Audit bucket | secure-s3-audit-teetech-20261021 |
| Region | us-east-1 |
| Log destination prefix | access-logs/ |
| CLI profile | personal-sandbox |
| Verified identity | ops-user |
| Test object | logging-test.txt |
| Test object encryption | SSE-KMS using alias/S3LabKey |
| Audit bucket encryption | SSE-S3 |
| Public access | Blocked on both buckets |

Actions Taken

Test Object ACL Modification

I attempted to apply a private ACL:

```powershell
aws s3api put-object-acl --bucket secure-s3-lab-teetech-20261021 --key confidential.txt --acl private --region us-east-1 --profile personal-sandbox
```

Actual response:

```text
AccessControlListNotSupported: The bucket does not allow ACLs
```

The bucket uses Bucket owner enforced Object Ownership, which disables ACLs. I retained this configuration and continued using policy-based access controls.

The ACL change was not applied.

 Create the Audit Bucket

Through the S3 console, I created:

```text
secure-s3-audit-teetech-20261021
```

Configuration:

- Region: us-east-1, matching the source bucket.
- Same AWS account as the source bucket.
- Block All Public Access enabled.
- ACLs disabled.
- SSE-S3 encryption.
- Versioning disabled.
- Server access logging disabled on the audit bucket itself.

Enable Logging on the Source Bucket

I navigated to:

S3 → secure-s3-lab-teetech-20261021 → Properties → Server access logging → Edit

I enabled logging and selected:

```text
s3://secure-s3-audit-teetech-20261021/access-logs/
```

I checked the destination policy for permission allowing the S3 logging service to deliver log objects.

 Generate Write Activity

```powershell
"Dummy data for server access logging verification." | Set-Content .\logging-test.txt
```

```powershell
aws s3 cp .\logging-test.txt s3://secure-s3-lab-teetech-20261021/logging-test.txt --sse aws:kms --sse-kms-key-id alias/S3LabKey --region us-east-1 --profile personal-sandbox
```

Actual output:

```text
upload: .\logging-test.txt to s3://secure-s3-lab-teetech-20261021/logging-test.txt
```

### Generate Read Activity

After resolving endpoint connectivity problems, I downloaded the object:

```powershell
aws s3 cp s3://secure-s3-lab-teetech-20261021/logging-test.txt .\logging-test-downloaded.txt --region us-east-1 --profile personal-sandbox --endpoint-url https://s3.us-east-1.amazonaws.com
```

Actual output:

```text
download: s3://secure-s3-lab-teetech-20261021/logging-test.txt to .\logging-test-downloaded.txt
```

This confirmed that the tested identity could download and decrypt the SSE-KMS object.

Verify Delivered Logs

Log files became visible in the audit bucket the following day.

I downloaded individual log objects through the console and collected them in a local PhaseD_AuditLogs folder.

From that folder, I searched their contents:

```powershell
Get-ChildItem -File -Recurse | Select-String -SimpleMatch "logging-test.txt"
```

The matching records confirmed the following activity.

| Request Time (UTC) | Operation | Object | HTTP Status | Interpretation |
|---|---|---|---|---|
| 21 October 2026, 21:08:09 | REST.HEAD.OBJECT | logging-test.txt | 404 NoSuchKey | Object requested before upload |
| 21 October 2026, 21:09:05 | REST.PUT.OBJECT | logging-test.txt | 200 | Successful upload |
| 21 October 2026, 21:12:54 | REST.HEAD.OBJECT | logging-test.txt | 200 | Successful metadata request |
| 21 October 2026, 21:12:55 | REST.GET.OBJECT | logging-test.txt | 200 | Successful download |

These are request timestamps from the log contents, not log-delivery timestamps.

Troubleshooting

| Issue | Resolution |
|---|---|
| Private ACL command rejected | Recorded the limitation and retained ACLs disabled |
| Download attempted before upload | Uploaded the local file before repeating the download |
| S3 endpoint connection and DNS failures | Checked connectivity and retried; the regional-endpoint download eventually succeeded |
| Audit bucket initially empty | Rechecked later and confirmed asynchronous delivery |
| Individual logs did not immediately show the test object | Collected multiple files and searched their contents |
| Local search initially used the wrong folder | Opened PowerShell in the actual log folder and repeated the search |

Key Observations

Bucket owner enforced prevented ACL changes. This was an expected restriction, rather than evidence that the object was public.

Delivered logs recorded successful upload and download operations, as well as the earlier missing-object request.

A HEAD operation checks metadata; it does not demonstrate retrieval of object contents. The successful GET record provided evidence of the download.

Server access logs arrived asynchronously. Delivery delay did not, by itself, establish a logging failure.

The successful SSE-KMS download extended the earlier encryption test by demonstrating decryption access for ops-user.

Evidence

- [ACL test result](PhaseD_ACL_Test.log)
- [Delivered log files](PhaseD_Logs.png)
- [Sanitized log verification output](PhaseD_Log_Verification.txt)

 Figure 1: Delivered Server Access Logs

![Log objects in the private audit bucket](PhaseD_Logs.png)

Log files delivered under access-logs/ in secure-s3-audit-teetech-20261021.

Completion Status

| Requirement | Result |
|---|---|
| Object ACL set to private | Not applied; ACLs disabled by Bucket owner enforced |
| Server access logging enabled | Completed |
| Log delivery verified | Completed |
| Successful upload recorded | REST.PUT.OBJECT — 200 |
| Successful download recorded | REST.GET.OBJECT — 200 |

Evidence Handling

Account identifiers, canonical owner IDs and personal IP addresses are redacted from public log evidence.

 References

- [S3 Object Ownership](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html)
- [Enabling S3 Server Access Logging](https://docs.aws.amazon.com/AmazonS3/latest/userguide/enable-server-access-logging.html)
