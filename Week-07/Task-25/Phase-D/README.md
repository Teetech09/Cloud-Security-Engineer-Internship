Phase D: Independent Extension, Policy Misconfiguration Analysis

Objective
The objective of this phase was to analyse the IAM policies in the lab account for privilege escalation potential and to rank the findings in a risk matrix. The target was the iam-vulnerable environment deployed in Phase B.

Actions Taken
The lab instructions used `aws iam list-policies` with a LocalStack endpoint. I adapted this in two ways. I removed the `--endpoint-url` flag because the environment runs in a real AWS lab account, and I replaced `list-policies` with `aws iam get-account-authorization-details`, because `list-policies` returns only policy names and ARNs, while Cloudsplaining needs the full policy documents. I saved the output to `policies.json`. The export took longer than expected because the account holds many users, roles, and policies, and the output is silent when it is redirected into a file.

I installed Cloudsplaining in a Python virtual environment and ran `cloudsplaining scan --input-file policies.json --output ./cloudsplaining_out`. The output folder had to exist before the scan, so I created it first. The scan produced an HTML report and two JSON files, `iam-results-policies.json` and `iam-findings-policies.json`. I then opened the HTML report in a browser and reviewed its summary and its Privilege Escalation findings. [FILL IN: summary counts from the report, such as the number of policies with privilege escalation, data exfiltration, and resource exposure findings.]

I mapped the findings into a risk matrix, scoring each one as likelihood multiplied by impact on a scale of 1 to 5, with MITRE ATT&CK references.

Key Observations
The findings fall into three groups. The first is full administrator access: `ops-user` and `Cloud-security-lab-admin` both have `AdministratorAccess`, so one leaked key would compromise the entire account. The second is permissions that let a user escalate: `fn1-privesc3-partial-user` has `iam:PassRole` on every resource, and the `privesc5` and `privesc6` users can create or change another user's console password. The third is low-risk access such as read-only S3 permissions. [FILL IN: one or two escalation methods that Cloudsplaining named for these policies, such as `CreateAccessKey` or `PassRole`.]

Cloudsplaining flagged the escalation paths from the policy text alone, without any login attempts, so it works well as a safe, read-only check. The report also shows that the combination of a broad allow and a narrow deny, as in the `fp2` user, is not exploitable, because explicit denies override allows in IAM. Not every flagged policy is equally urgent, which is why ranking by likelihood and impact matters.

Risk Matrix

| Cloud Asset / Resource | Threat / Risk | MITRE Ref. | Likelihood (1-5) | Impact (1-5) | Score | Rating |
|---|---|---|---|---|---|---|
| IAM users `ops-user`, `Cloud-security-lab-admin` (`AdministratorAccess`) | A leaked or guessed key gives full control of the account | T1078.004 Valid Accounts: Cloud Accounts | 4 | 5 | 20 | Critical |
| IAM user `fn1-privesc3-partial-user` (`iam:PassRole` on `*`) | Attacker passes a privileged role to a new service and escalates to its permissions | T1548 Abuse Elevation Control Mechanism | 3 | 5 | 15 | High |
| IAM users `privesc5-CreateLoginProfile-user`, `privesc6-UpdateLoginProfile-user` | Attacker sets a known password on another user and logs in as them | T1098 Account Manipulation | 3 | 5 | 15 | High |
| IAM user `cloudtrail-test-user` (`AmazonS3ReadOnlyAccess`) | Read-only data exposure if the key leaks | T1530 Data from Cloud Storage | 2 | 2 | 4 | Low |

Scores are likelihood multiplied by impact, with 1-7 rated Low, 8-14 Medium, 15-19 High, and 20-25 Critical.

Mitigations
Each high-rated finding maps back to least privilege. Administrator access should be replaced with scoped roles and short-lived credentials with MFA, `iam:PassRole` should be limited to specific role ARNs, and permissions to create or update login profiles should be restricted to a small administrative group.

Evidence Produced
The screenshot PhaseD_matrix.png shows the Cloudsplaining report summary alongside the completed risk matrix. The account ID has been blurred.
