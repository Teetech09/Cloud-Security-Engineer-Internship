 Phase E: Hosted Labs

Objective

The objective of this phase was to apply the techniques from the earlier phases in hosted, realistic environments by completing two Cybr labs: "IAM PutRolePolicy PrivEsc" and "IAM AttachUserPolicy PrivEsc".

Actions Taken

I reviewed both labs on the Cybr platform and found that each of them requires a premium subscription. I did not have access to a premium plan, so I could not launch the lab environments, and I did not complete either exercise. I did not substitute a different lab or claim any results from them.

Both labs cover escalation techniques that I had already worked on in the previous phases of this task. In Phase A, Cloudsplaining flagged `iam:PutRolePolicy` and `iam:AttachUserPolicy` as privilege escalation methods in my sample policy. In Phase C, PMapper identified the resulting escalation in my local environment, where `privesc-user` was treated as an administrative principal because of those permissions. In Phase D, I removed those permissions from the policy, denied them explicitly as a guardrail, and re-scanned with Cloudsplaining, which reported zero privilege escalation instances afterwards.

Key Observations

The two techniques follow the same idea from different directions. With `iam:AttachUserPolicy`, an identity attaches a more powerful managed policy, such as `AdministratorAccess`, to a user it controls, which could be its own user. With `iam:PutRolePolicy`, an identity writes a new inline policy onto a role it can use, which grants that role any permissions the attacker chooses. In both cases the identity does not need administrator rights to begin with, and the permission to change IAM is enough to reach them.

The mitigation I designed in Phase D applies to both techniques: IAM write permissions should not be given to everyday identities, they should be scoped to specific resources when they are needed, and an explicit deny can act as a guardrail. The Cloudsplaining re-scan is the closest thing I have to the mitigation confirmation this phase asks for, but it is not the same as completing the hosted labs, and I want to be clear about that difference. The labs would have tested the techniques against a live AWS environment, while my work in Phases C and D used a local simulator, which does not enforce IAM permissions.

Evidence Produced

The hosted labs could not be completed because of the premium requirement, so there is no PhaseE_Lab_result.png from them. 
