Week 07 – Phase A: Theory & Introduction
AWS STS AssumeRole and Weak Trust Relationships


Objective

Understand how AWS Security Token Service (STS) issues temporary credentials, and how attackers can exploit weak or overly permissive trust relationships.

Background

AWS STS lets an identity request short-lived credentials for an IAM role. Access depends on two things being true at the same time:

1. The role's trust policy** names the caller (or an account or service) as an allowed principal for `sts:AssumeRole`.
2. The caller's identity policy** permits `sts:AssumeRole` on that role. This second check applies when the caller and the role are in different accounts, or when the trust policy trusts the account root rather than a specific principal.

When both checks pass, STS returns:

| Field | Purpose |
|---|---|
| `AccessKeyId` | Temporary key ID (starts with `ASIA`, which marks temporary credentials) |
| `SecretAccessKey` | Secret paired with the key ID |
| `SessionToken` | Token that must accompany every request made with these credentials |
| `Expiration` | When the credentials stop working (default 1 hour) |

Lab Steps

Confirm the starting identity
```bash
aws sts get-caller-identity
```
Result: the caller was the IAM user `ops-user` in my account.

Create a trust policy
The policy trusts only the `ops-user` principal:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/ops-user" },
    "Action": "sts:AssumeRole"
  }]
}
```

Create the role and attach a harmless permission
```bash
aws iam create-role --role-name ExampleRole \
  --assume-role-policy-document file://trust.json

aws iam attach-role-policy --role-name ExampleRole \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

Assume the role
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/ExampleRole \
  --role-session-name testSession
```


Observations

- The returned credentials are **temporary** and expire automatically, unlike long-term IAM user access keys.
- The `AccessKeyId` begins with `ASIA`, which identifies it as a session credential.
- The session is attributed in logs to the **assumed-role ARN** and the session name (`testSession`), which is useful for tracing activity back to a specific session.
- All three values (key ID, secret, session token) are needed together. Using the key and secret without the token fails.

How These Credentials Can Be Abused When Trust Is Too Permissive

| Weakness | How an attacker exploits it |
|---|---|
| Wildcard principal (`"Principal": {"AWS": "*"}`) with no conditions | Any AWS account can assume the role, so an outsider who learns the role ARN gains access. |
| Trusting the account root with broad `sts:AssumeRole` grants** | Any compromised user or role in the account can pivot into a more privileged role (privilege escalation). |
| Missing `ExternalId` on third-party (vendor) roles** | Confused deputy attack: a vendor can be tricked into assuming a customer's role on the attacker's behalf. |
| No conditions (MFA, source IP, `aws:PrincipalOrgID`) | A stolen key from anywhere can assume the role without extra verification. |
| Role chaining | The attacker hops from role to role, getting fresh credentials each time and extending access past the original key's rotation or revocation. |
| Stolen session tokens | They work like normal credentials until expiry. Deleting the original IAM user does not invalidate sessions already issued. |
| Trust policy tampering | With `iam:UpdateAssumeRolePolicy`, an attacker adds their own principal to a role's trust policy as a persistence backdoor. |

6. Detection and Mitigation

Detection
- Monitor CloudTrail `AssumeRole` events for unusual source IPs, unexpected principals, new regions, or odd session names.
- Alert on changes to trust policies (`UpdateAssumeRolePolicy`, `CreateRole`).
- Use IAM Access Analyzer to flag roles that can be assumed externally.

Mitigation
- Trust specific principal ARNs instead of account roots or wildcards.
- Add conditions: `aws:MultiFactorAuthPresent`, `aws:SourceIp`, `aws:PrincipalOrgID`, `sts:ExternalId` for third parties.
- Apply least privilege to both the role's permissions and who may assume it.
- Keep session durations short and review roles regularly.

Clean-up

```bash
aws iam detach-role-policy --role-name ExampleRole \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam delete-role --role-name ExampleRole
```

Completion Checklist

- [x] Read STS documentation
- [x] Executed `assume-role` command successfully
- [x] Identified potential misuse scenarios
Evidence
    • PhaseA_sts_output.png —The screenshot shows the successful output of the aws sts assume-role command
      
Conclusion

STS makes cloud access safer by replacing long-lived keys with short-lived credentials, but the security of that model depends on the trust policy. A role that trusts too broadly turns any single compromised identity into a path to higher privileges. Tight trust policies, conditions, least privilege and CloudTrail monitoring close most of these paths.
