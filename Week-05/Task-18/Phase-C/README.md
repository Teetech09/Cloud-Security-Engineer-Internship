Phase C: Simulated Compromise & Detection

Objective

The objective of this phase was to simulate credential theft from the EC2 instance metadata service and to detect the resulting activity using logs.

Actions Taken

The task brief described reaching the vulnerable instance by SSH, but the `ec2_ssrf` scenario has no SSH key and is not built around direct instance access. Its own documentation describes a multi-stage privilege chain starting from the IAM user Solus, so I followed that route instead and adapted the brief's steps to match it.

Starting as Solus, I listed Lambda functions and found `cg-lambda-cgidzu8n53qwm5`, whose environment variables contained a second set of credentials, for a user referred to as Wrex. Configuring those as the `cglambda` profile, `aws sts get-caller-identity` confirmed the identity `wrex-cgidzu8n53qwm5`, and `aws ec2 describe-instances` revealed the target instance, `i-043033d482b3dc12a`, running a web application at its public IP.

The application was vulnerable to server-side request forgery through a `url` query parameter. Requesting `http://169.254.169.254/latest/meta-data/iam/security-credentials/` through the app returned the attached role name, `cg-ec2-role-cgidzu8n53qwm5`. Requesting that same path with the role name appended returned the role's temporary credentials in full: an access key, secret key and session token.

Setting these credentials up as a local profile proved to be the hardest part of this phase. The session token is a very long string, and several attempts to paste it into `nano` or directly onto the command line failed, either because the terminal split the paste across multiple lines or because the token's slashes were misread as shell syntax. By the time this was resolved, the first stolen credential set had expired, since these tokens are short-lived by design. I solved this by writing a single script that performs the SSRF request, parses the JSON response with `jq`, and sets each credential field on the `cgec2role` profile directly from the parsed values, removing the need to copy the token by hand at all.

With this profile working, `aws sts get-caller-identity --profile cgec2role` returned the assumed-role identity `arn:aws:sts::718543338561:assumed-role/cg-ec2-role-cgidzu8n53qwm5/i-043033d482b3dc12a`, confirming the stolen credentials were live. `aws s3 ls --profile cgec2role` then listed the account's S3 buckets, including `cg-secret-s3-bucket-cgidzu8n53qwm5`, which is the scenario's next step in its own privilege-escalation chain.

The brief's remaining step called for a GuardDuty finding named `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`, but GuardDuty is not available on this account, which is on the AWS Free account plan, as recorded in Phase B. In its place, I used CloudTrail directly. Using the stolen access key ID, `aws cloudtrail lookup-events` returned the `GetCallerIdentity` and `ListBuckets` calls made under that key, exported to `events.json`.

Key Observations

The actual exploitation route turned out to be more realistic than the task brief's SSH scenario, since it reflects how this kind of compromise typically happens in practice: an attacker pivoting through several sets of weaker credentials before reaching the metadata service, rather than logging directly into the instance. The hardcoded Lambda credentials and the exposed `url` parameter were both separate vulnerabilities that had to combine for the attack to reach the metadata service at all, which illustrates that IMDSv1 exposure is rarely the only flaw in a real compromise.

Handling a session token safely in a terminal turned out to be a practical skill in its own right. A token of this length is easy to corrupt through copy-paste, and using `jq` to parse it directly out of the API response removed that risk entirely, which is a better approach generally when working with any API-issued credential rather than a one-off fix.

The CloudTrail substitution still proved the detection objective. Both `GetCallerIdentity` and `ListBuckets` were logged under the stolen access key within minutes of the calls being made, each entry including the calling identity, source IP and time, which is the same information a GuardDuty finding would have been built from. The absence of GuardDuty means there is no automatic alert, so this activity would only be caught by someone actively reviewing CloudTrail or by a SIEM ingesting it, which is a meaningful detection gap compared with an account that has GuardDuty enabled.

Evidence Produced

The evidence for this phase is PhaseC_detection.png, a screenshot combining the SSRF request returning the role credentials, the `get-caller-identity` result confirming the assumed role, and the `s3 ls` bucket listing, and `events.json`, the CloudTrail export showing the `GetCallerIdentity` and `ListBuckets` events under the stolen access key. The generated credentials themselves were not included in any evidence file.
