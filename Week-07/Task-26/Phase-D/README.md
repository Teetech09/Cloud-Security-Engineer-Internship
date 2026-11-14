Phase D: Independent Extension & Mitigation Design

Objective

The objective of this phase was to develop mitigation strategies by editing the IAM policies from the previous phases so that the risky permissions are removed while the identities keep the access they legitimately need, and to confirm the improvement with a second Cloudsplaining scan.

Actions Taken

I started by identifying the root cause of each escalation path found in Phase C. The first path was `privesc-user`, which PMapper treated as an administrative principal. The cause was `RiskyDevPolicy`, which allowed `iam:PassRole`, `iam:AttachUserPolicy`, `iam:PutRolePolicy` and `iam:CreatePolicyVersion` on all resources, so the user could grant itself any permission. The second path was `LowPrivRole` reaching `AdminRole`. Its cause was a combination of two things: a policy that let `LowPrivRole` call `sts:AssumeRole` on `AdminRole`, and a trust policy on `AdminRole` that trusted the whole account instead of a named administrator.

The task brief suggests adding an explicit `Deny` on `iam:PassRole` for all resources. An explicit deny works, but it is a blunt tool, because it blocks legitimate uses as well and always overrides any allow. I preferred least privilege, which means removing or scoping the allow itself, and I kept a small explicit deny only as a guardrail. I saved the original policy as `phaseD_policy_before.json` and wrote a replacement as `phaseD_policy_after.json`. The new policy lets the user read objects from one named S3 bucket, describe EC2 instances, and pass a single named role, `AppRuntimeRole`, but only to the Lambda service through an `iam:PassedToService` condition. It denies `iam:AttachUserPolicy`, `iam:PutRolePolicy`, `iam:CreatePolicyVersion` and `iam:UpdateAssumeRolePolicy` on all resources. For the second path, the recommended fix is to change the trust policy on `AdminRole` so that it trusts one named administrator principal and requires multi-factor authentication, which stops `LowPrivRole` from assuming it.

To verify the policy change, I wrapped both versions of the policy in the account authorization details format that Cloudsplaining expects, using a small Python script, and ran `cloudsplaining scan` on each. This produced one report for the original policy and one for the revised policy.

Key Observations

The comparison between the two reports is shown below. The Severity column is the rating Cloudsplaining assigns to each risk category, and Instances is the number of findings in this policy.

| Risk | Before: instances | After: instances | Severity rating |
|---|---|---|---|
| Privilege Escalation | 1 | 0 | High |
| Data Exfiltration | 0 | 0 | Medium |
| Resource Exposure | 1 | 0 | High |
| Credentials Exposure | 0 | 0 | High |
| Infrastructure Modification | 1 | 0 | Low |

The original policy was flagged for privilege escalation, resource exposure, and infrastructure modification. After the revision, every category reported zero instances. Removing the IAM write actions eliminated the privilege escalation finding, and scoping `iam:PassRole` to one role, with a condition on the service it can be passed to, removed the exposure that came from allowing it on every resource. This matches what the earlier phases suggested: the danger came from the wildcard scope and from the IAM write permissions, not from any single harmless-looking action.

The revised policy still lets the user do the work a developer would reasonably need, so the change reduced risk without turning the identity into an unusable one. The explicit deny acts as a second layer, since it keeps these four actions blocked even if a broader allow is added to the user later.

There is a limit I want to record. Cloudsplaining reads a policy in isolation, so the re-scan confirms the fix for `RiskyDevPolicy` but cannot show that the `LowPrivRole` path is closed, because that path depends on the trust policy. Confirming it needs a new PMapper graph after the trust policy is changed, and the bucket and role names in the revised policy are placeholders that a real environment would replace with its own.

Evidence Produced

PhaseD_policy_before.json is the original `RiskyDevPolicy`, and PhaseD_policy_after.json is the least-privilege replacement. PhaseD_scan_before.png and PhaseD_scan_after.png are screenshots of the Cloudsplaining executive summary for each version, showing the drop in findings.
