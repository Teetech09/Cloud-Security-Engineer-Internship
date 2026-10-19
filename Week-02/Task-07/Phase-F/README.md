Phase F — Documentation and Reporting

Objective

Consolidate the IAM security assessment activities, findings, evidence, limitations, and recommendations from Phases A–E into a clear record.

Scope

The task covered IAM enumeration, CloudGoat scenario analysis, identity and permission mapping, visibility-gap analysis, and guided lab exercises.

Summary of Activities

| Phase | Activity |
|---|---|
| A | Reviewed IAM enumeration techniques and security misconfigurations. |
| B | Worked on the CloudGoat IAM scenario and its setup requirements. |
| C | Analysed IAM policies and mapped permissions to potential security risks. |
| D | Reviewed Access Analyzer results and assessed visibility gaps and risks. |
| E | Completed the Cybr exercise and partially completed the PwnedLabs exercise. |

Key Findings

1. Potential privilege escalation through IAM permissions

Policy analysis identified permissions associated with adding and removing roles from instance profiles and passing roles.

The Kerrigan policy included `iam:AddRoleToInstanceProfile`, `iam:RemoveRoleFromInstanceProfile`, and `iam:PassRole`.

These permissions warrant further review because their impact depends on resource scope, applicable conditions, available roles, and other permissions.

Security impact: If a usable escalation path exists, an attacker could obtain additional privileges and affect the confidentiality, integrity, or availability of accessible resources.

Validation status: A potential escalation path was identified through policy analysis. Exploitation was not demonstrated.

Recommendation: Restrict role-passing and instance-profile permissions to approved roles and resources, and review the complete permission combination.

2. Broad access identified during policy review

The Mighty policy allowed all actions in the reviewed policy, while Meek denied all actions.

Security impact: Broad permissions could increase the damage caused by compromised credentials. Actual access still depends on other applicable policies and controls.

Recommendation: Replace unnecessary wildcard permissions with permissions limited to required tasks and resources.

3. External-access findings did not resolve internal risks

The Access Analyzer review returned no external-access findings.

Observation: This result did not establish that all IAM permissions were secure or that potential access risks within the account had been eliminated.

Recommendation: Supplement external-access analysis with policy review, permission testing, and assessment of potential privilege-escalation paths.

Tools and Troubleshooting

- Used Policy Sentry through its Python API to produce `analysis.txt`.
- The attempted Policy Sentry CLI `analyze` command was unavailable in the installed version.
- Confirmed that John the Ripper could run.
- GoAWSConsoleSpray setup remained incomplete because Go was unavailable.
- PwnedLabs password recovery and subsequent AWS access were not verified as completed.

Evidence References

Supporting evidence is recorded in the Phase C, Phase D, and Phase E documentation.

The Phase D analysis is documented in:
`documentation/phaseD_visibility_risk_analysis.md`

Screenshots and outputs should be read alongside their captions and recorded testing limitations.

Outstanding Items

- Complete and attach the Phase D risk matrix.
- Complete the remaining PwnedLabs activities.
- Confirm that each finding has a matching evidence reference.
- Include the required control mapping in the final assessment report.

 Lessons Learned

IAM policy analysis can identify potentially dangerous permissions, but a potential attack path must be distinguished from successful exploitation.

Tooling problems and incomplete exercises should be documented clearly. Reporting these limitations accurately makes the assessment more useful and credible.

Conclusion

The assessment highlighted broad permissions, potential privilege-escalation risks, and the limits of relying only on external-access findings. Further validation and outstanding lab activities remain necessary.
