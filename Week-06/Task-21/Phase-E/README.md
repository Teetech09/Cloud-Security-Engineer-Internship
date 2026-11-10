Phase E: Investigating Threats with Amazon Detective

Project Overview

This phase involved investigating suspicious activities involving an Amazon EC2 instance using Amazon GuardDuty and Amazon Detective.

The practical exercise was conducted using the PwnedLabs lab, **Investigate Threats with Amazon Detective**.

The primary objective was to investigate an SSH brute-force alert, analyse suspicious authentication activity, correlate IP addresses, and determine whether the EC2 instance showed indicators of compromise.

Objectives

- Investigate GuardDuty SSH brute-force findings.
- Identify suspicious source IP addresses.
- Analyse EC2 instance activity using Amazon Detective.
- Examine Linux authentication logs.
- Correlate successful SSH logins with observed IP addresses.
- Assess potential post-compromise activity.

Tools and Technologies

- Amazon GuardDuty
- Amazon Detective
- Amazon EC2
- AWS CloudTrail
- VPC Flow Data
- Linux Authentication Logs (`auth.log`)
- PwnedLabs Cybersecurity Lab

Investigation and Actions Taken

GuardDuty Finding Analysis

Identified a GuardDuty finding:

- Finding type: `UnauthorizedAccess:EC2/SSHBruteForce`
- Severity:Low
- Region:`us-west-1`
- Affected instance: `i-0f9368c3dc7714c42`
- Source IP: `44.219.62.158`
- Destination port: 22 (SSH)

The finding indicated repeated SSH connection attempts against the EC2 instance.

Amazon Detective Investigation

Used Amazon Detective to investigate the affected EC2 instance, associated IP addresses, IAM entities, and resource interactions.

The investigation identified these IP addresses associated with the instance:

| IP Address | First Observed |
|---|---|
| `172.31.14.243` | 03 September 2023, 19:21 UTC |
| `13.57.2.249` | 03 September 2023, 21:51 UTC |

4.3 Linux Authentication Log Analysis

Reviewed the provided `auth.log` file and identified:

- 2,558 failed password authentication entries against `backupuser`.
- Successful SSH authentication for `backupuser` from `44.219.62.158` at 21:36:04 UTC.
- Successful SSH authentication for `backupuser` from `98.159.226.88` at 21:45:38 UTC.
- Successful SSH authentication for `backupuser` from `98.159.226.95` at 21:51:00 UTC.
- Successful public-key authentication for `ubuntu` from `98.159.226.95` at 22:20:34 UTC.

These events provided evidence that SSH authentication was successful from multiple external IP addresses.

5. Key Findings

1. Repeated SSH authentication failures targeted the `backupuser` account.
2. Multiple successful SSH authentication events followed the failed attempts.
3. Several external IP addresses were associated with successful authentication.
4. Amazon Detective provided additional context for examining the affected EC2 instance and its network activity.
5. The available evidence established suspicious access but did not fully establish the attacker's subsequent actions or the extent of any compromise.

Flag Retrieval Status

Status: Not Retrieved

I was unable to identify or retrieve a flag during this investigation. The available evidence supported the identification of suspicious SSH activity and successful authentication, but the investigation did not establish a complete post-compromise activity chain leading to a flag.

## 7. Limitations and Further Investigation

Further investigation would be required to establish the full impact of the suspicious access, including:

- Reviewing shell history and executed commands.
- Examining running processes and scheduled tasks.
- Investigating unusual outbound network connections.
- Reviewing IAM activity and potential privilege escalation.
- Investigating possible access to or exfiltration of S3 data.

Lessons Learned

This exercise strengthened my practical understanding of:

- Investigating SSH brute-force alerts using Amazon GuardDuty.
- Using Amazon Detective for security investigation and entity correlation.
- Analysing Linux authentication logs.
- Correlating timestamps and IP addresses across different evidence sources.
- Distinguishing suspicious authentication attempts from confirmed successful access.
- Documenting investigation findings and limitations accurately.

Evidence

Screenshots and supporting evidence are stored in the `Evidence` directory.

- PwnedLabs lab completion screenshot.
- Amazon Detective investigation screenshots.
- Investigation findings and authentication log evidence.

 Conclusion

The exercise demonstrated how Amazon GuardDuty and Amazon Detective can support cloud security incident investigation.

Although a flag was not retrieved, the investigation identified multiple successful SSH authentication events following repeated failed attempts. These findings highlighted the importance of correlating cloud telemetry with operating-system authentication logs when investigating potential EC2 compromise.
