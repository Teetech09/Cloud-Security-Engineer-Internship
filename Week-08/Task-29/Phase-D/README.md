Phase D: Core Execution — flaws.cloud Levels 4-6

Level 4 — Public EBS Snapshot Exposure

Actions Taken

Using the AWS credential recovered from Level 3 (profile `flaws-l3-leaked`), I attempted to read the Level 4 bucket directly but was denied `s3:ListBucket`. Reading `index.html` directly with `s3 cp` succeeded, since object-level `GetObject` access did not require bucket listing permission. The page explained that an EC2 instance's EBS volume had been snapshotted shortly after nginx was configured on it, and that the snapshot had been left publicly accessible.

I confirmed this by running `aws ec2 describe-snapshots` against the target account ID from my own sandbox profile, which returned a single public, unencrypted 8GB snapshot. Rather than working against it directly, I copied the snapshot into my own sandbox account, created a volume from that copy, and attached it as a secondary device to a newly launched, minimal `t3.micro` EC2 instance in the same region and availability zone. I mounted the volume read-only (using the `noload` option to skip ext4 journal replay, since the filesystem needed journal recovery) under `/mnt/l4snap`, so that no writes touched the original data.

Inspecting the mounted filesystem, I found an nginx configuration protecting the challenge's web content with HTTP basic authentication (`/etc/nginx/.htpasswd`), and a `setupNginx.sh` script in the home directory of the original `ubuntu` user containing the plaintext password used to generate that hash. I used this credential to authenticate to the live Level 4 EC2 webpage with `curl`, which confirmed completion and revealed the Level 5 URL.

Once confirmed, I unmounted the volume, detached and deleted it, terminated the EC2 instance, deleted the temporary security group and key pair (shredding the local private key file), and deleted the snapshot copy from my account. I verified cleanup by querying for any remaining instances, volumes, security groups, or snapshots tagged `Project=flaws-capstone` in the account, all of which returned empty after the instance showed a `terminated` state.

Key Observations

The account's leaked key could list the names of every bucket in the account, even though it lacked permission to list the contents of any individual bucket; AWS does not support restricting which specific bucket names a principal can see when listing, only whether they can access contents. This illustrates why bucket naming itself should not be treated as confidential: a globally unique S3 bucket name can be discovered even by a principal with no read access to it.

The core lesson of this level was that an EBS snapshot is, in effect, a complete forensic image of a volume, including configuration files, credential files, and shell history, and making one public exposes everything on that disk at the time it was taken, not just the specific file an administrator may have intended to share. The original administrator's own command history showed them repeatedly testing `htpasswd`, and ultimately leaving the working plaintext password sitting in a shell script. Their history also showed they had already manually queried the EC2 Instance Metadata Service and the `iam/security-credentials` path on that original instance, which foreshadows the SSRF credential theft that Level 5 covers.

Working safely with someone else's exposed resource required care at each step: copying the snapshot into my own account rather than acting on the original directly, keeping the inspection to the minimum resources needed (a single small instance, one volume, no larger footprint), mounting strictly read-only so the source data was never modified, and fully tearing down every temporary resource, with verification, before moving on, since EC2 and EBS resources accrue cost for as long as they exist.

Findings — Level 4

| AWS Service / API | Misconfiguration | Business Impact | MITRE ATT&CK for Cloud | Preventive Controls |
|---|---|---|---|---|
| EBS (`ec2:DescribeSnapshots`, snapshot permissions) | EBS snapshot was shared publicly rather than with specific accounts | Any AWS account holder can copy the snapshot and mount it to read the full disk contents, including configuration files, credentials, and shell history present at snapshot time | Credential Access: Unsecured Credentials; Collection: Data from Cloud Storage | (1) Never set EBS snapshot permissions to public; share explicitly with specific account IDs only, and audit snapshot permissions regularly; (2) encrypt EBS volumes and snapshots by default, so even an exposed snapshot cannot be read without the KMS key, and avoid leaving setup credentials or scripts containing plaintext secrets on disk |

Evidence Produced (Level 4)

- PhaseD_Level4_Intro.png — challenge intro page naming the target EC2 and the hint that a snapshot was taken shortly after nginx setup
- PhaseD_Level4_SnapshotFound.png — `describe-snapshots` output identifying the public snapshot
- PhaseD_Level4_MountSuccess.png — confirmation of the read-only mounted filesystem
- PhaseD_Level4_NginxConfig.png — nginx site configuration and `.htpasswd` entry
- PhaseD_Level4_BashHistory.png — original administrator's shell history, showing htpasswd setup attempts and early IMDS exploration
- PhaseD_Level4_Password.png — plaintext password recovered from `setupNginx.sh`
- PhaseD_Level4_Complete.png — authenticated request to the live Level 4 page confirming completion and revealing the Level 5 URL
- PhaseD_Level4_CleanupProof.png — verification that no tagged instance, volume, security group, or snapshot remains in the sandbox account



Level 5 — SSRF and EC2 Instance Metadata

Actions Taken

The Level 4 intro page's HTTP basic authentication had already revealed the Level 5 URL, and its nginx configuration had earlier shown an open proxy location (`/proxy/<host>/<path>`) with no destination allow-list. The Level 5 page confirmed this proxy was running on the same EC2 instance and gave benign usage examples.

I tested whether the proxy could be directed at the EC2 Instance Metadata Service by requesting `/proxy/169.254.169.254/`, since that address is only reachable from inside the instance's own network, not from my own machine directly. The proxy returned the standard list of supported metadata API versions, confirming the SSRF path was live and that the service responded without requiring any token (IMDSv1-style behaviour). I walked further into the metadata tree to `latest/meta-data/iam/security-credentials/`, which returned the name of the instance's attached IAM role, and then to that same path plus the role name, which returned temporary AWS credentials: an access key, secret key, and session token, all obtained purely through unauthenticated HTTP requests relayed by the proxy.

I configured these three values, including the session token, into a new named profile, since temporary STS credentials require the token in addition to the key pair. I verified the identity with `aws sts get-caller-identity`, which returned an assumed-role ARN tied to the specific EC2 instance that issued it. Using this credential, I performed the minimum enumeration needed: a recursive listing of the Level 6 bucket, which revealed a hidden subdirectory not visible in a plain top-level listing, and I read its index page to confirm the Level 6 entry point.

Key Observations

The IP address `169.254.169.254` is a fixed, well-known address used by AWS and other cloud providers to let an instance query its own metadata, including temporary credentials for whatever IAM role is attached to it. Because this address is not restricted to processes running directly on the instance, anything on the instance capable of making an outbound HTTP request to an arbitrary host, including a proxy with no destination restrictions, can be made to request it instead and relay the response back. This is a textbook Server-Side Request Forgery: the vulnerable component is not reachable to credential theft directly, but can be tricked into making the request on the attacker's behalf.

IMDSv2 would close this specific path, since it requires an initial `PUT` request carrying a custom header to obtain a session token before any metadata can be read, and a simple pass-through GET proxy like the one here has no mechanism to perform that handshake or forward the resulting header; without that header, IMDSv2 refuses the request outright. More broadly, restricting what hosts or IP ranges a proxy or any outbound-request feature is allowed to reach, specifically blocking link-local and other private ranges, would prevent this regardless of which IMDS version is in use. Finally, scoping the EC2 instance's IAM role to only the specific permissions it actually needs limits the damage even if credentials are stolen; in this case, the role was able to read the Level 6 bucket, which is more access than a simple nginx proxy server should ever require.

Findings — Level 5

| AWS Service / API | Misconfiguration | Business Impact | MITRE ATT&CK for Cloud | Preventive Controls |
|---|---|---|---|---|
| EC2 Instance Metadata Service (IMDSv1) via an unrestricted HTTP proxy | An application-level proxy with no destination allow-list enabled Server-Side Request Forgery against the EC2 Instance Metadata Service, and IMDSv1 served temporary role credentials with no token requirement | An attacker who can control a proxied or server-side-fetched URL on any application running on the instance can steal the IAM role's temporary credentials and act with whatever permissions that role holds | Credential Access: Unsecured Credentials (steal application access token); Initial Access / Privilege Escalation via SSRF to cloud instance credentials | (1) Enforce IMDSv2 account-wide (`HttpTokens: required` on the instance metadata options), which blocks simple GET-based SSRF from completing the required token handshake; (2) restrict any proxy, webhook, or server-side URL-fetching feature from reaching link-local (`169.254.0.0/16`) and other private address ranges, and apply least-privilege to the instance role so a stolen credential grants minimal access |

Evidence Produced (Level 5)

- PhaseD_Level5_Intro.png — Level 5 challenge page describing the proxy and the task
- PhaseD_Level5_SSRF_Confirmed.png — proxy successfully relaying a request to the EC2 metadata root
- PhaseD_Level5_RoleName.png — IAM role name retrieved via the metadata service
- PhaseD_Level5_Identity.png — `get-caller-identity` output confirming the assumed-role identity
- PhaseD_Level5_HiddenDir.png — recursive bucket listing revealing the hidden Level 6 prefix


Level 6 — IAM and Serverless Permission Discovery

Actions Taken

The challenge provided an access key directly, described as having the `SecurityAudit` managed policy attached. I configured it in its own named profile and verified the caller identity, confirming an IAM user named `Level6` in the target account. I enumerated the policies attached to this user and found two: a custom `MySecurityAudit` policy and a second custom policy named `list_apigateways`. Reading the latter's policy document directly, rather than guessing from its name, showed it granted `apigateway:GET` scoped to `arn:aws:apigateway:us-west-2::/restapis/*`, which covers actions on a specific, already-identified REST API's sub-resources but not the broader "list all REST APIs" collection action, confirmed when that broader call was denied.

To find the specific API ID this policy was meant to be used against, I used the user's broader `SecurityAudit`-derived read access to look for related resources elsewhere in the account. Checking Lambda functions revealed a function named `Level6`. Reading that function's resource-based policy showed a statement permitting the `apigateway.amazonaws.com` service to invoke it, with a condition restricting this to a specific source ARN containing the REST API's ID and the method and path (`GET /level6`) it was wired to. With that API ID in hand, calling `apigateway:GET` against that specific API's stages (an action the policy did allow) returned a single deployed stage. Combining the API ID, the stage name, and the path from the Lambda policy produced the final invoke URL, which I called to confirm completion of the challenge.

Key Observations

This level demonstrates the difference between a policy's name or stated intent and its actual effective permissions: a policy named for listing API Gateways did not in fact grant the IAM action needed to list all REST APIs in the account, because the AWS resource ARN for that specific action differs from the ARN pattern the policy actually matched. Reading the policy document directly, rather than trusting its name or description, is what reveals this gap, and it is precisely the kind of effective-permission analysis the task asked for rather than guessing or brute-forcing.

The path to the correct API ID ran through a different service entirely: a Lambda function's own resource-based policy, which is often overlooked as a source of information but reliably reveals what is allowed to invoke a function and under what source ARN conditions, directly exposing the identifiers needed elsewhere. A broad read-only policy such as `SecurityAudit` makes this kind of cross-service pivoting straightforward, since it is designed to let an auditor inspect configuration across nearly every service, which is also exactly why such a policy should be issued only to identities that genuinely need full-account visibility, and why administrators should assume that anyone holding it can map relationships between resources even without any explicit permission naming those relationships directly.

Findings — Level 6

| AWS Service / API | Misconfiguration | Business Impact | MITRE ATT&CK for Cloud | Preventive Controls |
|---|---|---|---|---|
| IAM / Lambda resource policy / API Gateway | A custom IAM policy's name implied broader access than its actual resource ARN granted, and the Lambda function's own resource-based policy exposed the API Gateway ID and route that located the hidden endpoint | A low-privilege, read-only identity could, through legitimate enumeration across IAM, Lambda, and API Gateway, fully reconstruct an API's internal identifiers and invoke a function never intended to be discovered this way | Discovery: Cloud Service Discovery; Discovery: Cloud Service Dashboard / Permission Groups Discovery | (1) Audit custom IAM policies against their actual resource ARNs rather than relying on policy names to convey scope, and avoid granting broad "audit" style access unless genuinely required; (2) treat resource-based policies (Lambda, S3, etc.) as sensitive configuration, since their `SourceArn` conditions can reveal internal identifiers, and avoid relying on obscurity (an unlisted API ID or stage) as the only protection for an endpoint |

Evidence Produced (Level 6)

- PhaseD_Level6_Intro.png — Level 6 challenge page with the provided access key and task description
- PhaseD_Level6_Identity.png — `get-caller-identity` confirming the `Level6` IAM user
- PhaseD_Level6_AttachedPolicies.png — attached policies, including `list_apigateways`
- PhaseD_Level6_PolicyDocument.png — the policy's actual JSON document and resource scope
- PhaseD_Level6_LambdaFound.png — the `Level6` Lambda function located via `lambda:ListFunctions`
- PhaseD_Level6_LambdaPolicy.png — the function's resource-based policy revealing the API Gateway ID and route
- PhaseD_Level6_StageFound.png — the deployed stage name retrieved from `apigateway:GET`
- PhaseD_Level6_Complete.png — confirmation response from the final endpoint (solution URL not reproduced in this report, per challenge scope)



Attack-Path Table: Levels 4–6

| Step | Credential / Permission Used | AWS API Call(s) | New Access Gained | Control That Breaks the Chain |
|---|---|---|---|---|
| 1 | Public EBS snapshot (no credential required beyond own AWS account) | `ec2:DescribeSnapshots`, `ec2:CopySnapshot`, `ec2:CreateVolume` | Read access to the full filesystem of a decommissioned EC2, revealing an administrator's setup script with a plaintext password | Never set snapshot permissions to public; restrict sharing to specific account IDs |
| 2 | Basic-auth password recovered from the snapshot | HTTP Basic Auth (`curl -u`) | Authenticated access to a live EC2 webpage running an HTTP proxy with no destination restrictions | Rotate/remove credentials discovered on any decommissioned resource; never leave setup scripts with plaintext secrets on disk |
| 3 | The open HTTP proxy itself (no AWS credential) | HTTP GET relayed to `169.254.169.254` (EC2 IMDS) | Temporary IAM role credentials (access key, secret key, session token) for the EC2 instance's attached role | Enforce IMDSv2 (`HttpTokens: required`); block proxy/SSRF access to the link-local metadata address range |
| 4 | Temporary role credentials (`assumed-role/flaws`) | `s3:ListBucket` (recursive) on the Level 6 bucket | Discovery of a hidden S3 prefix pointing to the Level 6 entry point | Apply least-privilege to the EC2 instance role; it should not need S3 read access to an unrelated bucket |
| 5 | Challenge-provided `Level6` IAM user credentials | `iam:ListAttachedUserPolicies`, `iam:GetPolicy`, `iam:GetPolicyVersion` | Identification of a custom policy's true effective scope (`apigateway:GET` on a specific, not-yet-known REST API) | Name and scope custom IAM policies to match their actual resource ARNs; review policies for over-broad or misleading grants |
| 6 | Same `Level6` credentials, broader `SecurityAudit`-style read access | `lambda:ListFunctions`, `lambda:GetPolicy` | The target REST API's ID and invocation path, read from the Lambda function's resource-based policy | Treat resource-based policies as sensitive configuration; restrict broad read/audit access to identities that genuinely need account-wide visibility |
| 7 | Same `Level6` credentials, now scoped correctly | `apigateway:GET` on the specific REST API's stages | The deployed stage name, completing the information needed to build the final invoke URL | Do not rely on an unlisted or undocumented API ID/stage as a security boundary; apply explicit authorization on the API method itself |

 Command Log

- PhaseD_Level4-6_CommandLog.txt — full terminal session for Levels 4 through 6, with all live credentials (the Level 5 SSRF-derived session token and the Level 6 provided access key/secret) redacted before submission and verified clean of key and token patterns.
