Task 21: AWS CloudTrail Security Monitoring

Phase D: Independent Extension – Log Integrity Validation

Objective

Validate that the CloudTrail logs delivered to Amazon S3 have not been modified or deleted by using CloudTrail digest files and the `aws cloudtrail validate-logs` command.

Environment

| Item | Value |
|---|---|
| Trail | `SecurityTrail` (multi-Region, log file validation enabled) |
| Log bucket | `cloudtrail-securitylogs-titilayo` |
| Region | `us-east-1` (US East – N. Virginia) |
| Tool | AWS CLI v2 in WSL Ubuntu, using the `ops-user` IAM user |
| Validation range | From `2026-11-09T00:00:00Z` (UTC) |

Actions Taken

I opened the S3 bucket `cloudtrail-securitylogs-titilayo` and located the digest files in the `AWSLogs/<account-ID>/CloudTrail-Digest/us-east-1/2026/11/09/` folder. The task text refers to `/AWSLogs/Digest/`, but CloudTrail actually stores digest files in a `CloudTrail-Digest` folder, next to the `CloudTrail` folder that holds the log files. Digest files are written about once an hour when log file validation is enabled, and I had enabled it when I created the trail in Phase B.

I then retrieved the trail's ARN with the AWS CLI:

```bash
aws cloudtrail describe-trails --region us-east-1 \
  --query 'trailList[?Name==`SecurityTrail`].TrailARN' --output text
```

I used the ARN to run the validation command. The `validate-logs` command takes `--trail-arn` rather than the trail name given in the task instructions:

```bash
aws cloudtrail validate-logs \
  --trail-arn <TRAIL_ARN> \
  --start-time 2026-11-09T00:00:00Z \
  --region us-east-1 \
  --verbose


The command downloads the digest files and log files from S3, verifies the digital signature on each digest, and checks the hash of each log file against the value recorded in the signed digest. I then captured the output as evidence.

Key Observations

The validation completed successfully:

```
5/5 digest files valid
69/69 log files valid
```

- All digest files were valid.Each digest's signature and its link to the previous digest in the chain were verified.
- All log files were valid. The SHA-256 hash of every log file in S3 matched the hash recorded in the signed digest, so none had been modified or deleted in the validated range.
- No invalid or missing files were reported.

Differences from the task text:
- Digest files are stored under `CloudTrail-Digest/`, not `/AWSLogs/Digest/`.
- The CLI uses `--trail-arn`, not `--trail-name`.
- The command prints a summary of valid file counts, rather than the words "Validation succeeded". All counts valid is the success result.

Note on timing:A digest is only produced after the trail has been delivering logs for a while, so the validation range has to start after log file validation was enabled and include at least one completed digest period.

How Digest Files Protect Against Tampering

- Hashing: Each hour, CloudTrail creates a digest file listing the log files delivered in that period, together with a SHA-256 hash of each one. If a log file is altered, its hash no longer matches the one in the digest.
- Digital signature: CloudTrail signs each digest file with a private key. Anyone can verify the signature with the matching public key, but nobody without the private key, including someone with access to the bucket, can create a valid digest for altered logs.
- Hash chain: Each digest contains the hash of the previous digest. Deleting or changing a digest breaks the chain, so removed or rewritten history can be detected.
- Detection of deletions: If a log file listed in a digest has been removed from S3, `validate-logs` reports it as missing.

Limitations: Log file validation detects tampering, but it does not prevent it. To protect log integrity in a real environment, the bucket should have tightly restricted write and delete permissions, S3 versioning, MFA Delete or S3 Object Lock, and ideally log delivery to a separate, dedicated security account. Alerts on changes to the trail, such as `StopLogging` or `DeleteTrail`, are also important, because an attacker may try to disable logging instead of altering the logs.

Security Relevance

Logs are only useful as evidence if they can be trusted. During an incident, an attacker with access may try to delete or edit CloudTrail records to hide their activity. Digest-based validation gives investigators and auditors a way to confirm that the records are complete and unmodified. This supports incident investigation, audit evidence, and compliance requirements that call for tamper-evident logging.

 Completion Checklist

- [x] Digest file located.
- [x] Validation command executed successfully.
- [x] Integrity verification documented.

Evidence Produced



- PhaseD_CloudTrail_Log_Validation_CLI_Output.png`: CLI output showing `5/5 digest files valid` and `69/69 log files valid`.
- PhaseD_S3_Digest_Folder.png: the `CloudTrail-Digest` folder in the S3 bucket, showing the digest files.

> Account ID and the trail ARN were blurred in the screenshots.

Conclusion

This phase confirmed that the logs delivered by `SecurityTrail` are intact. Running `validate-logs` verified every digest and log file in the range, and showed how signed hashes and the digest chain make tampering detectable. It also showed that log file validation is a detective control, so it should be combined with preventive controls on the log bucket and alerts on attempts to disable logging.

References

- [Validating CloudTrail log file integrity](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-intro.html)
- [Validating CloudTrail log file integrity with the AWS CLI](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-cli.html)
- [CloudTrail digest file structure](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-digest-file-structure.html)
- [Enabling log file integrity validation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-enabling.html)
