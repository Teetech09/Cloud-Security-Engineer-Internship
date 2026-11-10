Task 21: AWS CloudTrail Security Monitoring
Phase F: Documentation & Reporting

Objective

The objective of this phase was to compile the findings from Phases A to E into a professional incident report. The report needed to present the work in a structured format that a reviewer could follow without having to read each phase separately.

Actions Taken

I reviewed the work completed in Phases A to E and gathered the evidence produced in each one. This included the CreateUser event JSON from Phase A, the SecurityTrail configuration and the S3 Region folders from Phase B, the annotated CloudTrail events from Phase C, the log integrity validation result from Phase D, and the GuardDuty, Detective and authentication log findings from Phase E.

I then organised this material into a single report named CloudTrail_Report_Titilayo.pdf. The report contains an executive summary, scope and methodology, findings and evidence for each phase, a risk matrix, recommendations, an evidence index and a conclusion. For the Phase E scenario, I stated a clear verdict, built a timeline of the successful SSH logins, and recorded the limitations of the investigation, including that the lab flag was not retrieved because of time constraints and that the second lab, which was premium, was not attempted.

Key Observations

CloudTrail recorded identity-changing events and denied API calls with enough detail to reconstruct what happened, including the calling identity, source, Region and time. Log file validation passed in full, with 5 of 5 digest files and 69 of 69 log files valid, which supports the use of these logs as audit evidence.

In the hosted lab, the EC2 instance was assessed as likely compromised. The source IP behind the brute-force finding later authenticated successfully, further logins followed from other external IP addresses, and a public-key login to the ubuntu account at 22:20 UTC suggested movement to a second account and possible persistence. GuardDuty rated the finding as Low severity, which understated the real outcome and showed why alerts should be correlated with logs and behavioural data rather than judged on severity alone.

Writing the report also showed the value of stating conclusions plainly and recording what was not established, so that a reader knows both what the evidence supports and where further investigation is needed.

Evidence Produced

The evidence for this phase is the final report
