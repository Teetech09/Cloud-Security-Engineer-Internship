Phase F: Documentation & Reporting

Objective

The objective of this phase was to compile a professional incident response report summarizing the full detection-to-recovery workflow carried out across Phases A through E of this task.

 Actions Taken and Observations, Phase A through E

 Phase A: Theory & Threat Modeling

Action Taken. I reviewed how the EC2 Instance Metadata Service exposes an instance role's temporary credentials at `169.254.169.254` under IMDSv1, where a single unauthenticated GET request returns the access key, secret key and session token. I compared this with IMDSv2, which requires a session token obtained through a PUT request before any metadata call succeeds. I built a threat model identifying the EC2 instance and its IAM role as the asset, credential theft as the threat, IMDSv1 exposure as the vulnerability, and IMDSv2 enforcement, WAF rules, least privilege and monitoring as mitigations, then completed a risk matrix scoring each threat by likelihood and impact.

Observation. The threat model showed that IMDSv1 turns a single application flaw, such as SSRF, into a full credential compromise, which scored the highest risk in the matrix at 20 out of 25. Enforcing IMDSv2 does not fix an application's own vulnerabilities, but it removes the metadata service as a reachable target for them, dropping that risk's residual score to 4. The remaining risks, an over-permissioned role and the application flaw itself, stayed at Medium even after mitigation, which established early that no single control would close every gap in this environment.

Phase B: Environment Setup

Action Taken. I deployed the CloudGoat `ec2_ssrf` scenario in `us-east-1`, resolving an SSH timeout on the first attempt by refreshing CloudGoat's IP whitelist and redeploying. I recorded the deployed resources: EC2 instance `i-043033d482b3dc12a`, IAM role `cg-ec2-role-cgidzu8n53qwm5`, and confirmed via `describe-instances` that the instance's `HttpTokens` metadata option was set to `optional`. I attempted to enable GuardDuty and Security Hub but both returned `SubscriptionRequiredException`, which the AWS console attributed to the account being on the Free account plan; I chose not to upgrade.

Observation. The `HttpTokens: optional` setting confirmed the environment matched exactly the vulnerability identified in the Phase A threat model, meaning the scenario was ready for exploitation. The GuardDuty and Security Hub limitation meant this environment would have no automated detection layer for the phases that followed, making CloudTrail the sole detection source for the rest of the task.

### Phase C: Simulated Compromise & Detection

**Action Taken.** Since the scenario had no SSH access, I followed its actual privilege chain instead of the task brief's SSH-based instructions: hardcoded credentials in a Lambda function's environment variables led from the low-privileged user Solus to a second user, Wrex, whose access revealed an EC2 instance running a web application vulnerable to SSRF. Exploiting the SSRF via its `url` parameter, I retrieved the EC2 instance role's temporary credentials directly from the metadata service. After resolving some local issues with handling the long session token, I configured these stolen credentials as an AWS CLI profile and used them to call `sts get-caller-identity` and `s3 ls`. In place of the unavailable GuardDuty finding, I used `aws cloudtrail lookup-events` to pull the matching log entries for the stolen access key.

Observation. The attack required two separate weaknesses, not just the IMDSv1 exposure: hardcoded Lambda credentials and an unvalidated SSRF parameter. CloudTrail fully substituted for the missing GuardDuty alert, logging both the `GetCallerIdentity` and `ListBuckets` calls made with the stolen credentials, complete with timestamp, source IP and user agent, though without an automated alert this activity would only surface through active log review.

 Phase D: Containment & Forensic Analysis

Action Taken. Since the stolen credential was a temporary session token rather than a deactivatable IAM key, I contained it by attaching an inline deny policy to the compromised role, `cg-ec2-role-cgidzu8n53qwm5`, blocking any session issued before the policy's application time. I then rotated the long-term credentials actually exposed earlier in the chain, for the IAM users Wrex and Solus, deactivating each old key after confirming a new one was active. For forensic review, in place of host-level logs that weren't reachable without SSH access, I used the Phase C CloudTrail export, extracting event names, timestamps, source IPs and user agents into `phaseD_iocs.txt`, and packaged the raw export alongside a filtered summary into `phaseD_logs.zip`. I also built a risk matrix covering the task's key findings, mapped to MITRE ATT&CK techniques.

Observation. Session credentials and long-term credentials require different containment actions, a distinction that matters in real incident response. The recorded source IP matched my own, confirming this was a self-contained simulation rather than a distinct external actor. Rotating Wrex's and Solus's keys closed the actual root cause of the chain, since a repeat SSRF attack would no longer find valid upstream credentials.

Phase E: Hosted Labs Extension

Action Taken. I completed the PwnedLabs lab "Loot Public EBS Snapshots" to reinforce the same category of exposure, credential theft through a misconfigured AWS resource, using a different attack surface. Starting from a low-privileged `intern` user, I confirmed a specific EBS snapshot was public and unencrypted, then restored it to a volume in my own AWS account, attached it to an EC2 instance, and mounted it. This revealed hardcoded AWS credentials in an application file on the disk, which I used to access a private S3 bucket and retrieve the lab's completion flag.

Observation. This lab reinforced that a public EBS snapshot exposes an entire disk image, not a single file, which is what made a credential buried in an old application file retrievable at all. It confirmed the same underlying lesson as Phases A through D from a different angle: credentials should never be hardcoded in files that could end up copied, backed up, or shared beyond their original context, whether that file lives in a Lambda function, an application source file, or anywhere else.

Summary: Detection-to-Recovery Workflow

Across this task, a single root cause, IMDSv1 left enabled on an EC2 instance, was modeled as a threat in Phase A, deployed and confirmed in Phase B, exploited through a realistic multi-stage privilege chain in Phase C, detected through CloudTrail in the absence of GuardDuty, and contained and remediated in Phase D through session revocation and credential rotation. Phase E reinforced the same class of vulnerability, hardcoded credentials exposed through a misconfigured AWS resource, in an independent environment, confirming the lesson generalizes beyond the EC2 metadata service specifically. The workflow followed the standard incident response lifecycle: identify, protect, detect, respond and recover, each phase's evidence file supporting that stage of the process.

Evidence Produced

This phase references the evidence already produced in Phases A through E: `phaseA_threatmodel.png`, `phaseB_resources.png`, `phaseC_detection.png` and `events.json`, `phaseD_iocs.txt` and `phaseD_logs.zip`, and `phaseE_lab.png`. No new evidence files were generated in this phase; its output is this consolidated report.
