Phase D: Containment & Forensic Analysis

Objective

The objective of this phase was to contain the incident from Phase C by revoking the compromised credentials and to perform a forensic review of the activity using the logs already collected.

Actions Taken

The stolen credential from Phase C was a temporary STS session token (`ASIA2OTEFHRA5AMAAWK2`), not a long-term IAM access key, so it could not be deactivated with `aws iam update-access-key`, which only applies to `AKIA`-type keys. Instead, I revoked the session at its source by attaching an inline deny policy, `RevokeOldSessions`, to the compromised role `cg-ec2-role-cgidzu8n53qwm5`. The policy denies all actions for any session whose `aws:TokenIssueTime` is earlier than the moment the policy was applied, which invalidates the stolen token and any other session issued before that point, while leaving the instance able to request fresh sessions normally.

I then rotated the two long-term credentials that were actually exposed earlier in the attack chain. For the IAM user `wrex-cgidzu8n53qwm5`, whose keys were found hardcoded in the Lambda function's environment variables in Phase C, I created a new access key and deactivated the old one, `AKIA2OTEFHRAYVB6XV26`. I did the same for `solus-cgidzu8n53qwm5`, deactivating `AKIA2OTEFHRAQAT2LKSQ`. Both were confirmed with `aws iam list-access-keys`, showing the old key as `Inactive` and the new key as `Active` in each case.

For the forensic review, the brief asked for a memory dump and system logs from the instance, but the scenario provides no SSH access, as established in Phase C, so there was no way to collect host-level artifacts. I used the CloudTrail export from Phase C as the forensic source instead, since it is the authoritative record of everything the stolen credentials did. Running `jq` against `events.json` produced a clean timeline of the two calls made with the stolen token, `GetCallerIdentity` and `ListBuckets`, each with a timestamp, and extracting the `sourceIPAddress` and `userAgent` fields from the raw CloudTrail event gave the indicators of compromise recorded in `phaseD_iocs.txt`.

Key Observations

The source IP recorded against both events, `105.113.90.198`, matches my own public IP address, confirmed with a direct check. This is expected in a self-contained lab where the same person plays both attacker and investigator, but it is worth stating plainly: these indicators would not distinguish an external attacker from legitimate activity in this specific case, since there is no second party involved.

The containment approach also highlighted a difference between session credentials and long-term credentials that matters in real incident response. A stolen session token cannot be revoked individually, it has to be cut off either by waiting for its natural expiry or by denying all sessions issued before a point in time, which also invalidates any other active sessions on that role. Rotating the long-term keys further up the chain, Wrex and Solus, was the part of containment that actually removes the attacker's ability to start the chain again, since a new SSRF exploitation attempt would no longer find working credentials in the Lambda environment.

The CloudTrail-based review covered everything the scenario's own objective asked for, API-level evidence of the compromise, but it has a real limitation compared with host-level forensics: it shows what the stolen credentials did in AWS, not how the SSRF payload was constructed or whether the instance itself was altered. In a real incident, both would be needed, and the absence of instance access here would be a gap I would flag rather than something to route around silently.

Evidence Produced

The evidence for this phase is PhaseD_iocs.txt, listing the compromised role, instance, stolen access key, the two long-term keys rotated, the session revocation action taken, and the CloudTrail entries with source IP and user agent for each; and `phaseD_logs.zip` (or `phaseD_logs.tar.gz`), containing the raw `events.json` export and a filtered summary of event names and timestamps. No access key secrets were included in either file.
