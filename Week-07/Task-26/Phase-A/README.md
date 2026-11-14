Phase A: Theory & Introduction

Objective

The objective of this phase was to understand how IAM privilege escalation occurs through misconfigurations such as `iam:PassRole`, `iam:AttachUserPolicy` and `iam:PutRolePolicy`, and to use Cloudsplaining to identify these risky permissions in a sample policy.

Actions Taken

I began by reviewing the concept of chained permissions, where one permission that looks harmless on its own lets an attacker obtain a higher-level permission, and then another, until they reach administrative access. I then studied the common escalation techniques: attaching an administrator policy to one's own user, injecting an inline policy with `iam:PutRolePolicy`, publishing a new default policy version with `iam:CreatePolicyVersion`, editing a role's trust policy with `iam:UpdateAssumeRolePolicy`, and passing a privileged role to a service such as EC2 or Lambda with `iam:PassRole`.

Next I installed Cloudsplaining 0.9.1 with pipx in WSL Ubuntu. Since Docker Desktop is the default WSL distribution on my laptop, I opened Ubuntu explicitly with `wsl -d Ubuntu`. I created a sample policy file named `sample-policy.json`, in the account authorization details format that Cloudsplaining expects, containing a deliberately risky policy called `RiskyDevPolicy` attached to a sample user, `sample-dev-user`. The policy allowed `iam:PassRole`, `iam:AttachUserPolicy`, `iam:PutRolePolicy`, `iam:CreatePolicyVersion`, `iam:UpdateAssumeRolePolicy`, `sts:AssumeRole`, `ec2:RunInstances`, `lambda:CreateFunction` and `s3:GetObject` on all resources. The file used placeholder account data only, so no real AWS information was involved.

I validated the JSON, created a `results` folder, and ran `cloudsplaining scan --input-file sample-policy.json --output ./results`. I then opened the generated HTML report in my browser, reviewed the executive summary, and went into the Customer Policies section to examine the findings for `RiskyDevPolicy` in detail.

Key Observations

The Cloudsplaining report for `RiskyDevPolicy`, which was attached to the sample user `sample-dev-user`, flagged four risk categories for this policy: Credentials Exposure, Data Exfiltration, Resource Exposure and Privilege Escalation. The executive summary rated Privilege Escalation, Resource Exposure and Credentials Exposure as High, Data Exfiltration as Medium and Infrastructure Modification as Low. The policy also reported nine infrastructure modification actions across five services, which shows how much of the AWS environment this one policy can change.

The Privilege Escalation section explained that the policy allows a combination of IAM actions that let a principal escalate its own privileges, for example by creating an access key for another IAM user or by modifying its own permissions. It listed the specific escalation methods that apply to this policy. These include `iam:CreatePolicyVersion`, where a user writes a new administrator version of a policy and makes it the default, `iam:AttachUserPolicy`, where a user attaches a powerful policy such as `AdministratorAccess` to themselves, and `iam:PutRolePolicy`, where a user adds an inline policy with wide permissions to a role. It also listed `iam:UpdateAssumeRolePolicy`, which lets a user change a role's trust policy so they can assume it, and the combination of `iam:PassRole` with `ec2:RunInstances`, which lets a user launch an EC2 instance with a more privileged role and then take that role's credentials. The report credits this research to Spencer Gietzen at Rhino Security Labs.

This showed me how chained permissions work in practice. None of these actions needs to be administrator access on its own. Each one gives the user a way to grant themselves a stronger permission, and that stronger permission can then be used for the next step. Because every action in the sample policy was allowed on all resources, a single compromised credential for `sample-dev-user` could lead to full control of the account. The remedy is least privilege: removing IAM write permissions from everyday identities, scoping `iam:PassRole` to specific role ARNs, and limiting policy changes to a small set of trusted administrators.

Evidence Produced

PhaseA_report.png is a screenshot of the Cloudsplaining report for `RiskyDevPolicy`, showing the Privilege Escalation findings and the specific methods flagged, including `iam:CreatePolicyVersion`, `iam:AttachUserPolicy`, `iam:PutRolePolicy`, `iam:UpdateAssumeRolePolicy` and `iam:PassRole` with `ec2:RunInstances`. `phaseA_report_summary.png` is a screenshot of the report's executive summary, showing the risk categories and their severity ratings.
