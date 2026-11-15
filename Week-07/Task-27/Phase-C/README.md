Week 07 – Phase C: Core Execution
STS Abuse Simulation — AssumeRole Chaining

Objective

Demonstrate how an attacker can exploit weak trust relationships to escalate privileges via AssumeRole chaining — using one set of temporary credentials to obtain another, hopping between roles.

Setup

Two roles were used to build the chain:

| Role | Trust Policy Principal | Permissions |
|---|---|---|
| `ExampleRole` | IAM user `ops-user` | `ReadOnlyAccess` |
| `RoleB` | IAM role `ExampleRole` | `ReadOnlyAccess` |

`RoleB`'s trust policy trusts the **role** `ExampleRole` itself, not a specific user, session, or external condition. That single line is the vulnerability the whole exercise demonstrates.

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "AWS": "arn:aws:iam::<ACCOUNT_ID>:role/ExampleRole" },
    "Action": "sts:AssumeRole"
  }]
}
```

Lab Steps

Extract the target role's trust policy
```bash
aws iam get-role --role-name RoleB | jq '.Role.AssumeRolePolicyDocument'
```
Confirmed the policy shown above: any session operating as `ExampleRole` is automatically trusted to become `RoleB`.

First assumption — `ops-user` → `ExampleRole`
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/ExampleRole \
  --role-session-name chainStep1
```
Result: new temporary credentials issued, `AssumedRoleUser` ARN `assumed-role/ExampleRole/chainStep1`.

Switch to those credentials and confirm
```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
aws sts get-caller-identity
```
Confirmed identity: `arn:aws:sts::<ACCOUNT_ID>:assumed-role/ExampleRole/chainStep1`.

Chained assumption — `ExampleRole` → `RoleB`
Using the `ExampleRole` session's own credentials (not the original `ops-user` credentials):
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/RoleB \
  --role-session-name chainStep2
```
Result: a second, independent set of temporary credentials, `AssumedRoleUser` ARN `assumed-role/RoleB/chainStep2`.

Confirm the final hop
```bash
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
aws sts get-caller-identity
```
Confirmed identity: `arn:aws:sts::<ACCOUNT_ID>:assumed-role/RoleB/chainStep2` — two hops away from the original `ops-user` identity.

Evidence file

All steps were logged and combined into a single JSON file, with the trust policy and both `assume-role` results:

```json
{
  "target_role_trust_policy": { ... },
  "chain": [
    { "step": 1, "action": "ops-user assumes ExampleRole", "result": { ... } },
    { "step": 2, "action": "ExampleRole assumes RoleB", "result": { ... } }
  ]
}
```
See `phaseC_assume_chain_redacted.json` for the full output (secrets and session tokens redacted; `AccessKeyId` values are shown since they only identify a credential and do not authenticate with it).

Key Observations

- Escalation succeeded purely because of the trust policy, not because of a software bug. `RoleB` trusted a role ARN with no further conditions, so possessing valid `ExampleRole` credentials was sufficient to become `RoleB` — no password, MFA, or additional approval required.
- Each hop produced independent, fresh credentials with their own `AccessKeyId`, `SecretAccessKey`, `SessionToken` and expiry. This is what allows chaining: an attacker holding only the first hop's credentials can mint new, equally valid credentials for the next role.
- The `AssumedRoleUser` ARN changes at each hop and records the session name used (`chainStep1`, `chainStep2`). This is the main trail available to a defender — CloudTrail logs each `AssumeRole` call with the calling identity, the target role, and the session name, which is what would let an analyst reconstruct this chain after the fact.
- No long-term credentials were ever exposed.The entire chain happened using only temporary, expiring tokens — which is exactly why trust-policy design matters more than key rotation in this scenario: rotating keys does nothing to stop a trust-policy-based escalation path.

Conditions That Allowed Escalation

| Condition | Effect |
|---|---|
| Trust policy names a role ARN, not a specific session or external ID | Any valid session for that role, however obtained, can assume the next role |
| No MFA condition (`aws:MultiFactorAuthPresent`) on the trust policy | Stolen or phished credentials are enough; no second factor is checked at assume-time |
| No source IP or `aws:PrincipalOrgID` condition | The assumption can happen from anywhere, not just expected networks |
| No `sts:ExternalId` requirement | There's no shared secret the caller needs to prove (relevant mainly for cross-account/third-party trust, but the lack of any constraint here is the same underlying issue) |

Mitigations

- Scope trust policies to the **narrowest principal possible** — a specific role session (via `sts:RoleSessionName` conditions) rather than any session of a role.
- Require `aws:MultiFactorAuthPresent: true` on sensitive roles.
- Add `aws:SourceIp` or `aws:PrincipalOrgID` conditions to restrict where assumptions can originate.
- Set maximum session durations as short as practical, so a leaked credential has a short useful life.
- Alert on `AssumeRole` call chains in CloudTrail — specifically sessions where the calling principal is itself an `assumed-role` ARN rather than a long-term user, which signals chaining.
- Apply IAM Access Analyzer to flag roles with external or overly broad trust

  Clean-up

```bash
aws iam detach-role-policy --role-name RoleB --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam delete-role --role-name RoleB
aws iam detach-role-policy --role-name ExampleRole --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam delete-role --role-name ExampleRole
```

Completion Checklist

- [x] Extracted trust policy JSON
- [x] Successfully assumed at least one chained role (two hops completed)
- [x] Logged all session tokens used (captured in `phaseC_assume_chain_redacted.json`; all have since expired)

Conclusion

This exercise showed that AssumeRole chaining is not an exploit against AWS itself, but against **how trust is configured**. A single overly permissive trust policy — trusting a role rather than a tightly scoped principal with conditions — let a chain of two hops move from a low-privilege user to a second role with no additional authentication. Defending against this means auditing every role's trust policy for scope and conditions, not just its permissions policy, and watching CloudTrail for assumed-role identities that go on to assume further roles.
