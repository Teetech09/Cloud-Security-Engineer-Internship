Phase F: Documentation & Reporting

Objective

The objective of this phase was to compile the findings from Phases A to E into a professional report that summarises the privilege escalation paths discovered, the risks, the mitigations and the business impact, with recommendations aligned to NIST SP 800-53 access control (AC) requirements.

Actions Taken

I wrote a seven-page report, `IAM_PrivEsc_Report.pdf`, that brings the whole task together. It begins with an executive summary of the two escalation paths found in the simulated AWS account, followed by a section on scope and methodology that lists each phase, the tools used and the output produced. The findings section describes four findings in detail: the user `privesc-user` that could escalate itself to administrator through IAM write permissions in `RiskyDevPolicy`, the role `LowPrivRole` that could assume the administrative role `AdminRole`, the unscoped `iam:PassRole` permission, and the Neo4j lab database that was running with authentication disabled. I supported these with the PMapper query output and with my Cloudsplaining screenshots from Phase A.

I then built the risk matrix with the columns the task asks for: asset, threat, likelihood, impact and risk level. I scored each risk as likelihood multiplied by impact on a scale of 1 to 5 and also drew a likelihood-versus-impact grid so that the position of each finding is easy to see. The mitigation section compares the original policy with the least-privilege version from Phase D, includes the Cloudsplaining before-and-after results, and gives the expected residual risk for each finding. I added a business impact analysis covering confidentiality, integrity, availability, compliance and reputation, and a table of recommendations mapped to NIST SP 800-53 controls: AC-2, AC-3, AC-5, AC-6 and its enhancements AC-6(5), AC-6(9) and AC-6(10), together with IA-2(1) for multi-factor authentication on privileged roles. The report ends with a limitations section and an index of the evidence from each phase.

Key Observations

The two escalation paths came from different causes but have the same effect. The first was a policy that gave a user permission to change IAM itself, and the second was a trust policy that let a low-privilege role reach an administrative one. In both cases nothing looked like administrator access when read on its own, and the danger appeared only when the permissions were combined. This is why mapping the paths with PMapper and scoring them was more useful than reading individual policies.

The risk matrix put the self-escalating user at 20 out of 25, which is Critical, the role chaining at 15 and the unscoped `iam:PassRole` at 12, both High, and the lab database at 4, which is Low. After the policy was rewritten to follow least privilege, the Cloudsplaining re-scan reported zero privilege escalation, resource exposure and infrastructure modification findings, down from one each. The recommendations are low cost, because they are policy edits and not new tooling, which makes least privilege one of the best controls available for this type of risk.

I stated the limitations in the report plainly. The Moto simulator does not enforce IAM permissions, so the paths were identified by analysis rather than exploited. The narrowed trust policy for `AdminRole` is a recommended design that I did not re-test with PMapper, so its residual score is an expected value. The two hosted Cybr labs in Phase E require a premium subscription and were not completed, so the Phase D re-scan is the closest available confirmation of the mitigation. The next steps are to apply the trust policy change and re-run PMapper, and to repeat the assessment in a real lab account, where permissions are enforced.

Evidence Produced

PhaseE_IAM_PrivEsc_Report.pdf is the final report, which includes the executive summary, methodology, findings, the risk matrix, the mitigation comparison, the business impact analysis, the NIST SP 800-53 recommendations, the limitations and an appendix listing the artefacts from each phase.
