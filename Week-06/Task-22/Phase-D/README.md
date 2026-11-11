Objective

The objective of this phase was to correlate IAM events into an attacker timeline and to build a risk matrix that rates each detected anomaly by likelihood and impact, with recommended mitigations.

Actions Taken

I exported the IAM events (CreateUser, AttachUserPolicy and PutUserPolicy) and the requests from the top S3 source IP from Athena to a CSV file, and loaded it into Python with pandas. I plotted a timeline of the IAM events by identity and event type using matplotlib, covering February 2017 to August 2020. I then built a likelihood-versus-impact risk matrix with six risks, scoring each from 1 to 5 on both axes, and combined the timeline and matrix into a single visual saved as PhaseD_visuals.png. Finally, I paired each risk with a mitigation based on least privilege, MFA and monitoring.

Key Observations

The timeline showed that IAM privilege activity was concentrated in two identities, Level6 (17 users created, 9 policy attachments, 2 inline policies) and backup (15 users created, 12 policy attachments, 4 inline policies). The earliest events in the export had no resolvable user name and came from an anonymised source address, and 19 events overall could not be attributed to a named user. Repeated user creation followed by policy changes forms a persistence and privilege-escalation pattern, so these were rated the highest risks. The high-volume S3 access from a hosting-provider IP and the lack of S3 data event logging were rated high and medium respectively, because the first indicates possible automated access and the second means actual data access cannot be confirmed. The likelihood and impact scores are my own judgement based on the log evidence, not measured values.

Risk Matrix and Mitigations
ID	Risk	Likelihood	Impact	Rating	Mitigation
R1	Mass IAM user creation (Level6, backup)	4	5	High	Restrict iam:CreateUser to named admins, enforce MFA, alert on CreateUser
R2	Policy attachment and inline policy escalation	4	4	High	Least-privilege review, permissions boundaries, alert on AttachUserPolicy and PutUserPolicy
R3	High-volume S3 access from a hosting IP	4	4	High	Restrict bucket access by IP or VPC endpoint, enable GuardDuty, block if unauthorised
R4	IAM actions with no resolvable user name	3	3	Medium	Enforce named roles and session naming for attribution
R5	No S3 data event logging	5	3	Medium	Enable CloudTrail data events for sensitive buckets
R6	Anonymised or unverifiable source IPs	2	2	Low	Retain original IPs and enrich logs with threat intelligence

Evidence Produced
    • PhaseD_Athena_Export_Query.png-Screenshot of Athena export 
    • PhaseD_Script_Output.png-Screenshot of Script output 
    • PhaseD_CSV_Visuals. png- Screenshot showing the IAM attacker timeline and the risk matrix. The Athena export (events.csv) and the analysis script (phaseD.py) are included in the repository for reproducibility.
