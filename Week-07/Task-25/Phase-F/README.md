Phase F: Documentation & Reporting

Objective
The objective of this phase was to compile the findings from Phases A to E into a professional security report on AWS IAM weaknesses, suitable for review by a technical or management audience.

Actions Taken
I reviewed the evidence and notes from every earlier phase and consolidated them into a single PDF report, `CSE_AWS_IAM_Report_TitilayoYusuf.pdf`. The report begins with an executive summary that explains, in plain language, how attackers gain initial access through leaked keys, weak passwords, and over-permissive IAM policies, and how least privilege limits the damage. It then describes the methodology: the Phase A theory on Initial Access in MITRE ATT&CK for Cloud, the Phase B deployment of the iam-vulnerable environment with Terraform, the Phase C enumeration of users and policies and the ScoutSuite scan, the Phase D Cloudsplaining analysis, and the Phase E hosted lab on IAM enumeration.

The findings section presents the main results with supporting screenshots: the users with full administrator access, the wildcard `iam:PassRole` policy, the users able to create or change other users' passwords, and the privilege escalation paths that Cloudsplaining reported. These are ranked in the Phase D risk matrix, which uses likelihood multiplied by impact on a 1 to 5 scale and maps each finding to a MITRE ATT&CK technique. The report closes with recommendations and an appendix that indexes all evidence files.

Key Observations
Writing the report showed how the phases fit together. Theory (Phase A) predicted the risks, the lab environment (Phase B) made them concrete, enumeration and scanning (Phases C and D) found them, and the risk matrix turned the technical findings into priorities. The most important lesson was that the account's highest risks came from permissions rather than software flaws: administrator access on long-lived users, wildcard policies, and permissions that allow credential manipulation. All of these are addressed by least privilege, MFA, short-lived credentials, and regular policy review. I also noted the limitations of the work: the lab instructions assumed LocalStack, so I adapted the commands to a real lab account, and the PwnedLabs lab in Phase E was not attempted due to time constraints.

Evidence Produced
The deliverable is the final PDF report, `CSE_AWS_IAM_Report_TitilayoYusuf.pdf`. It includes screenshots from all phases (`phaseA_summary.png`, `phaseB_setup.png`, `phaseC_enum.png`, `phaseC_report.png`, `phaseD_matrix.png`, and `phaseE_labproof.png`) with account IDs blurred.
