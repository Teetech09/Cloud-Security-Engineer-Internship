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

Actions Taken

1. Accessed the PwnedLabs **Investigate Threats with Amazon Detective** lab environment and examined the available AWS security resources in the `us-west-1` region.

2. Opened Amazon GuardDuty and reviewed the SSH brute-force finding:
   - Finding type: `UnauthorizedAccess:EC2/SSHBruteForce`
   - Affected EC2 instance: `i-0f9368c3dc7714c42`
   - Source IP: `44.219.62.158`
   - Destination port: `22 (SSH)`

3. Opened Amazon Detective and investigated the affected EC2 instance, its associated IP addresses, IAM user, IAM role, and related finding groups.

4. Examined the EC2 instance's Overview, Resource interaction, and VPC Flow information to understand its observed network activity and associated IP address assignments.

5. Investigated suspicious IP addresses, including `44.219.62.158`, `98.159.226.88`, and `98.159.226.95`, and compared their observed activity with the authentication log timestamps.

6. Downloaded and examined the provided Linux `auth.log` archive. Reviewed authentication records for failed and successful SSH login attempts involving the `backupuser` and `ubuntu` accounts.

7. Identified successful SSH authentication events and correlated their timestamps with the IP addresses observed in Amazon Detective.

8. Adjusted the Amazon Detective investigation scope to 3 September 2023, 21:36–22:41 UTC, to focus on the period containing the identified successful SSH logins.

9. Reviewed the available evidence and documented the investigation findings, limitations, and flag retrieval status.

Key Observations

The investigation revealed that the affected EC2 instance experienced repeated SSH authentication failures targeting the `backupuser` account. The authentication log contained 2,558 failed password authentication entries, followed by multiple successful SSH logins from external IP addresses.

The successful login events were particularly significant because they demonstrated that the activity went beyond unsuccessful authentication attempts:

- `44.219.62.158` successfully authenticated as `backupuser` at 21:36:04 UTC.
- `98.159.226.88` successfully authenticated as `backupuser` at 21:45:38 UTC.
- `98.159.226.95` successfully authenticated as `backupuser` at 21:51:00 UTC.
- `98.159.226.95` later authenticated as `ubuntu` using a public key at 22:20:34 UTC.

The correlation between the authentication log and Detective's IP address observations provided useful evidence for reconstructing the incident timeline.

One important lesson was that a GuardDuty SSH brute-force finding alone does not establish successful compromise. Reviewing operating-system authentication logs provided stronger evidence of successful access.

Although Amazon Detective helped establish relationships between AWS resources and network entities, the available investigation data did not fully establish what actions were performed after authentication or whether sensitive information was accessed or exfiltrated.

Investigation outcome: Suspicious SSH activity and successful external authentication were identified. However, I was unable to establish the complete post-compromise activity chain or retrieve the lab flag. Further investigation of system processes, shell history, outbound connections, IAM activity, and S3 access would be necessary to determine the full impact.
Tools and Technologies

- Amazon GuardDuty
- Amazon Detective
- Amazon EC2
- AWS CloudTrail
- VPC Flow Data
- Linux Authentication Logs (`auth.log`)
- PwnedLabs Cybersecurity Lab



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

Key Findings

1. Repeated SSH authentication failures targeted the `backupuser` account.
2. Multiple successful SSH authentication events followed the failed attempts.
3. Several external IP addresses were associated with successful authentication.
4. Amazon Detective provided additional context for examining the affected EC2 instance and its network activity.
5. The available evidence established suspicious access but did not fully establish the attacker's subsequent actions or the extent of any compromise.

Flag Retrieval Status

Status: Not Retrieved

I was unable to identify or retrieve a flag during this investigation. The available evidence supported the identification of suspicious SSH activity and successful authentication, but the investigation did not establish a complete post-compromise activity chain leading to a flag.

 Limitations and Further Investigation

Further investigation would be required to establish the full impact of the suspicious access, including:

- Reviewing shell history and executed commands.
- Examining running processes and scheduled tasks.
- Investigating unusual outbound network connections.
- Reviewing IAM activity and potential privilege escalation.
- Investigating possible access to or exfiltration of S3 data.


Evidence Produced
    • GuardDuty_SSH_Brute_Force_Finding.png—GuardDuty Detection of SSH Brute-Force Activity Against EC2 Instance
    • GuardDuty_SSH_BruteForce.png —This screenshot shows an Amazon GuardDuty security finding identifying SSH brute-force activity against the EC2 instance i-0f9368c3dc7714c42 in the us-west-1 region. The finding identifies 44.219.62.158 as the source IP address and classifies the activity as UnauthorizedAccess: EC2/SSHBruteForce with Low severity. This finding served as the starting point for further investigation in Amazon Detective, where I examined the affected instance, associated IP addresses, and suspicious authentication activity.



Lessons Learned

This exercise strengthened my practical understanding of:

- Investigating SSH brute-force alerts using Amazon GuardDuty.
- Using Amazon Detective for security investigation and entity correlation.
- Analysing Linux authentication logs.
- Correlating timestamps and IP addresses across different evidence sources.
- Distinguishing suspicious authentication attempts from confirmed successful access.
- Documenting investigation findings and limitations accurately.



 Conclusion

The exercise demonstrated how Amazon GuardDuty and Amazon Detective can support cloud security incident investigation.

Although a flag was not retrieved, the investigation identified multiple successful SSH authentication events following repeated failed attempts. These findings highlighted the importance of correlating cloud telemetry with operating-system authentication logs when investigating potential EC2 compromise.
