Phase D: Risk Assessment & Threat Modeling

Objective
Build a threat model showing how an attacker could exploit exposed S3 data, using the misconfigured environment from [Phase B](../Phase-B) (CloudGoat `cloud_breach_s3`, which created a cardholder data bucket) as the reference scenario.

Evidence
| File | Description |
|------|-------------|
PhaseD_risk_matrix.pdf | Risk matrix, threat model diagram, heat map and mitigation recommendations (4 pages) |

Actions Taken
1. **Identified assets:** Customer Data Bucket, Logs Bucket, web-facing EC2 instance with its IAM role, and IAM roles/users/keys.
2. **Built the risk matrix** mapping each asset to its threats, consequences, likelihood and impact (Low / Medium / High), resulting risk rating and mitigation. Eight threats were assessed (T1 to T8).
3. **Drew the threat model diagram** showing two attacker paths to the Customer Data Bucket, with mitigation controls placed on each path:
   - **Path A:** misconfigured reverse proxy, then SSRF to the instance metadata service, then stolen IAM role credentials, then S3 data download.
   - **Path B:** direct access through a public ACL or permissive bucket policy.
4. **Plotted a likelihood vs impact heat map** and ranked the mitigations by priority.

Risk Summary
| ID | Asset | Threat | Risk |
|----|-------|--------|------|
| T1 | Customer Data Bucket | Public ACL / bucket policy exposure | High |
| T2 | Web-facing EC2 + IAM role | SSRF to metadata service, credential theft | High |
| T3 | IAM role / permissions | Over-permissive S3 access (e.g. `s3:*` on `*`) | **Critical** |
| T4 | Customer Data Bucket | Bulk download goes undetected | High |
| T5 | Logs Bucket | Log deletion or tampering | Medium |
| T6 | Logs Bucket | Logs readable by too many principals | Low |
| T7 | Customer Data Bucket | No encryption at rest / shared key | Medium |
| T8 | IAM users / access keys | Leaked long-lived access keys | High |

Recommended Mitigations (priority order)
1. Least-privilege IAM: scope roles to the specific bucket ARN and actions; add permissions boundaries.
2. Enable S3 Block Public Access at account and bucket level; remove public ACLs.
3. Enforce IMDSv2 and harden the reverse proxy against SSRF.
4. Detective controls: CloudTrail data events, GuardDuty S3 protection, Macie, alarms.
5. Encrypt with SSE-KMS and require TLS through the bucket policy.
6. Protect the logs: separate account, Object Lock, log file validation.
7. Remove static keys: use roles/SSO, rotate keys, scan repos for secrets.

Key Observation
The highest risk (T3, Critical) is excessive IAM permission, because it decides how much an attacker gains once any credential is obtained. Block Public Access, IMDSv2 and least-privilege roles together remove or lower most of the High-rated threats. Ratings are qualitative judgments for a lab exercise, not measured probabilities.

 Completion Checklist
- [x] Risk matrix created
- [x] Threat model diagram completed
- [x] Mitigation recommendations documented
