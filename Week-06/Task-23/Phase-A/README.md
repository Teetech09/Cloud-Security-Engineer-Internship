Phase A: Theory & Methodology Overview

Cloud Incident Response Methodology for AWS Environments

Objective

This phase aimed to explain the NIST SP 800-61r2 incident response lifecycle and how it adapts to AWS cloud environments. The phase focused on the purpose of each of the four lifecycle phases, the AWS services that support each one, and how an alert travels from a detection service to the people who respond to it.

Actions Taken

I began by reviewing the four phases of the NIST SP 800-61r2 lifecycle: Preparation, Detection and Analysis, Containment, Eradication and Recovery, and Post-Incident Activity. For each phase I identified its purpose and then mapped it to the AWS services and actions that support it. Preparation maps to least-privilege IAM, a multi-Region CloudTrail with log file validation, GuardDuty and Security Hub, SNS contact lists and documented runbooks. Detection and Analysis maps to GuardDuty findings, CloudTrail events queried in Athena, CloudWatch alarms, Amazon Detective and AWS Config. Containment, Eradication and Recovery maps to IAM policy lockdown, disabling access keys, isolation with security groups or SCPs, EBS snapshots for forensics, secret rotation and restoring from clean backups. Post-Incident Activity maps to rebuilding the timeline from CloudTrail and Athena, tuning detection rules, reviewing access with IAM Access Analyzer and updating runbooks.

I then created two diagrams. The first shows the four lifecycle phases with their AWS services and a feedback loop from Post-Incident Activity back to Preparation. The second shows the alert flow GuardDuty to EventBridge to Lambda to SNS. GuardDuty produces a finding, an EventBridge rule matches it by source and severity, a Lambda function enriches and triages it and can take a pre-approved containment action, and an SNS topic notifies the response team. CloudTrail and CloudWatch Logs record each step so that the later timeline has evidence to draw on. I combined the diagrams, the phase-to-service table and a summary paragraph for each phase into one PDF.

Key Observations

The main observation is that cloud incident response depends on preparation more than on any single tool. Detection services, logging and notification paths have to be in place before an incident, because responders cannot create logs or permissions retroactively. The lifecycle is also a cycle, since lessons from Post-Incident Activity feed back into Preparation.

In AWS, containment is often an identity action and not a network action. Attaching a deny policy, disabling an access key or revoking active sessions can stop an attacker quickly, which is why IAM is central to both detection and response. Automation through EventBridge and Lambda can shorten the time between a finding and a response, but automated containment should be limited to well-tested, low-risk actions, with an analyst confirming anything uncertain. Evidence such as snapshots and logs should be preserved before resources are changed or deleted.

This phase was theory and methodology only. The alert flow shown is a reference design and was not deployed.

Evidence Produced

The evidence for this phase is PhaseA_IRFlow.pdf. It contains the lifecycle diagram mapped to AWS services, the GuardDuty to EventBridge to Lambda to SNS flow diagram, a phase-to-service mapping table and a summary paragraph explaining the purpose of each phase.
