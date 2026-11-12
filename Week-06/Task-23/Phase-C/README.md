PHASE C - AI-ASSISTED ATTACK CHAIN SUMMARY
Cloud Incident Response Methodology for AWS Environments
Scenario: CloudGoat iam_privesc_by_attachment (cgidjij1nc9chc)

NOTE ON METHOD: The task brief's original workflow asks for GuardDuty findings to be reviewed and then summarized by an AI tool into MITRE ATT&CK attack stages. GuardDuty could not be enabled on this AWS account (Free account plan, SubscriptionRequiredException on CreateDetector), as already documented in Phase B. In its place, the attacker's actions
were reconstructed directly from AWS CloudTrail events via "aws cloudtrail lookup-events", and the same AI-assisted MITRE ATT&CK mapping step was applied to those events instead.


ATTACK CHAIN SUMMARY


Stage 1 - Initial Access & Identity Verification (09:37 UTC+1)
The attacker authenticated as the low-privilege IAM user "kerrigan" (credentials supplied by the CloudGoat scenario, simulating a leaked or phished access key) and confirmed the identity with GetCallerIdentity. MITRE ATT&CK: T1078.004 - Valid Accounts: Cloud Accounts

Stage 2 - Discovery (09:38-09:39 UTC+1)
The attacker attempted to enumerate her own IAM policies, which was denied, then pivoted to enumerating IAM roles and instance profiles account-wide, successfully identifying a low-privilege role (cg-ec2-meek-role) and a high-privilege role (cg-ec2-mighty-role).
MITRE ATT&CK: T1087.004 - Account Discovery: Cloud Account
             T1526 - Cloud Service Discovery

Stage 3 - Privilege Escalation (09:39-09:41 UTC+1)
A direct attempt to create a new instance profile was denied. The attacker then abused her permission to modify an EXISTING instance profile: she removed the meek role from cg-ec2-meek-instance-profile and attached the mighty role in its place. This re-purposed a profile she was allowed to touch into a vehicle carrying higher privileges.
MITRE ATT&CK: T1098.003 - Account Manipulation: Additional Cloud Roles

Stage 4 - Execution & Credential Theft via Compute Resource (09:45 UTC+1 onward)
The attacker launched an EC2 instance with the now-escalated instance profile attached. AWS automatically assumed cg-ec2-mighty-role on behalf of the instance. By connecting to the instance (SSH) and querying its Instance Metadata Service (IMDSv2), the attacker retrieved temporary credentials for the mighty role directly, bypassing the need for any
further IAM API calls. GetCallerIdentity run with these credentials confirmed the assumed role: arn:aws:sts::718543338561:assumed-role/cg-ec2-mighty-role-cgidjij1nc9chc/i-0e7fe695da62fad3a
MITRE ATT&CK: T1578.002 - Modify Cloud Compute Infrastructure: Create Cloud Instance
             T1552.005 - Unsecured Credentials: Cloud Instance Metadata API

Stage 5 - Persistence (11:55-12:00 UTC+1)
The original exploitation instance was terminated (routine lab cleanup / OPSEC), and a second instance was launched using the same escalated instance profile, with an attacker-imported SSH key pair for repeatable access. This confirms the privilege escalation is persistent at the IAM configuration level (the instance profile/role binding) and does not depend on any single compute instance surviving.
MITRE ATT&CK: T1578.002 - Modify Cloud Compute Infrastructure: Create Cloud Instance

KEY PATTERN FOR DETECTION

No single API call in this chain is inherently malicious - ListRoles, RunInstances, and AddRoleToInstanceProfile are all routine administrative actions. The anomaly is the SEQUENCE and the IDENTITY: a user with no self-read IAM permissions successfully enumerating roles account-wide, then modifying an instance profile's role binding, then immediately launching an instance with that profile, all within a four-minute window (09:37-09:41) from a single source IP (105.113.103.235). This pattern - discovery, instance-profile manipulation, then compute launch - is the signature CloudGoat "iam_privesc_by_attachment" escalation path and should be the basis for a detection rule (e.g., alert when AddRoleToInstanceProfile or RemoveRoleFromInstanceProfile is followed within minutes by RunInstances using the same profile, from an identity that lacks iam:Get*/iam:List* permissions on itself).


LIMITATION

Because GuardDuty was unavailable, this summary relies entirely on CloudTrail management events. CloudTrail records the API calls that constitute the escalation, but it does not independently flag them as suspicious the way a GuardDuty finding (e.g., PrivilegeEscalation:IAMUser/AnomalousBehavior) would. In a production environment with GuardDuty enabled, this same CloudTrail evidence would typically also surface as one or more automated findings, which would then feed the EventBridge -> Lambda -> SNS pipeline documented in Phase A.

Evidence Produced
    • PhaseC_Findings.json — four categorized findings (IAM discovery, instance-profile privilege escalation, compute-based credential theft, persistence across instance replacement), each with MITRE ATT&CK technique IDs and CloudTrail timestamps. 
    • PhaseC_Summary.txt — the AI-generated attack chain summary, with the adapted prompt documented and the GuardDuty limitation noted. 
    • PhaseC_InstanceProfileSwap.png — Terminal capture of the list-instance-profiles output before and after the role swap, confirming the escalation. 
    • PhaseC_AssumedRoleIdentity.png — Terminal capture of aws sts get-caller-identity run from inside the EC2 instance, showing the assumed-role ARN.
