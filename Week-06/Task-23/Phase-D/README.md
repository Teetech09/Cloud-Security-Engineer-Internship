Phase D: Containment, Eradication & Recovery

Cloud Incident Response Methodology for AWS Environments

Objective

The objective of this phase was to execute containment actions through IAM lockdown and automated remediation. The task brief’s original workflow asks for an EventBridge rule triggered by GuardDuty findings. Since GuardDuty remains unavailable on this account (Free account plan, documented in Phase B), the rule was built to trigger on the actual CloudTrail event that drove the privilege escalation in Phase C: AddRoleToInstanceProfile.

Actions Taken

I built a Lambda function, ContainCompromisedUser, with an execution role (LambdaContainmentRole) scoped to only iam:ListAccessKeys and iam:UpdateAccessKey on the user kerrigan, plus the permissions needed to write to CloudWatch Logs. The function reads the triggering event, extracts the username, and deactivates all of that user’s access keys.

I created an EventBridge rule, DetectInstanceProfilePrivesc, matching CloudTrail AddRoleToInstanceProfile events from aws.iam, granted it permission to invoke the Lambda, and attached the Lambda as its target. I confirmed the rule was ENABLED and correctly targeted with aws events describe-rule and aws events list-targets-by-rule.

To test the pipeline, I re-triggered the escalation from Phase C: as kerrigan, I swapped the instance profile’s role from mighty back to meek, then back to mighty again. Both AddRoleToInstanceProfile calls matched the rule and invoked the Lambda. I confirmed execution by retrieving the Lambda’s CloudWatch Logs, saved as PhaseD_LambdaLogs.txt.

For recovery, I terminated the exploitation EC2 instance from Phase C (i-0e7fe695da62fad3a) and restored the instance profile to its original state by removing the mighty role and reattaching meek, confirmed with aws iam list-instance-profiles.

Key Observations

The containment pipeline worked exactly as designed: both test invocations completed in under 350ms with no errors, and the Lambda’s log output clearly shows the triggering CloudTrail event, the target username, and the specific access key it deactivated. The second invocation deactivating an already-inactive key confirms the function is idempotent, which matters for a real deployment, since the same alert could plausibly fire more than once for related activity.

Scoping the Lambda’s permissions to a single named user, rather than a wildcard resource, was a deliberate least-privilege choice: if the Lambda itself were ever compromised or misconfigured, its blast radius is limited to one account rather than the whole IAM user base. A production version of this function would take the username from the event dynamically rather than defaulting to a single hardcoded user, which is a limitation of this lab implementation worth noting.

Triggering on the raw CloudTrail event (AddRoleToInstanceProfile) instead of a GuardDuty finding has a real trade-off. GuardDuty would typically wrap this kind of behavior in a correlated, severity-scored finding (for example PrivilegeEscalation:IAMUser/AnomalousBehavior), after comparing it against a baseline of normal activity. My rule fires on every occurrence of this specific API call, with no behavioral baselining, which means it would also fire on a legitimate administrator rotating an instance profile’s role as part of routine maintenance. This is a reasonable substitute for demonstrating the containment mechanism, but it is not equivalent to GuardDuty’s detection quality, and I have noted this gap rather than presenting the substitution as a like-for-like replacement.

Recovery in this task was not a snapshot restoration, since no data-bearing resource was compromised; the EC2 instance here was the attacker’s own exploitation host, not a victim system. Recovery instead meant terminating that instance and reverting the IAM misconfiguration it depended on, which is the realistic recovery action for this specific incident type.

Evidence Produced
PhaseD_Containment.png — the EventBridge rule’s state (ENABLED) and its target, showing the Lambda ARN wired to the rule.
PhaseD_LambdaLogs.txt — CloudWatch Logs from both test invocations, showing the triggering CloudTrail event, the containment action taken, and successful completion with no errors.

Checklist status:

EventBridge rule created and tested: met
Lambda function executed successfully: met
Recovery validation complete: met
