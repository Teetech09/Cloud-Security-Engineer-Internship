Phase F – Documentation & Reporting

Objective

Compile the results from Phases A–E into a professional report explaining IAM architecture decisions, findings, risks, and security mitigations.

Report Summary

The final report brings together:

- Executive Summary: Overview of the work, key findings, and outcomes.
- Methodology IAM setup, CLI enumeration, policy analysis, simulation, and guided labs.
- Findings & Risks: Broad S3 read permissions, credential-related risks, and the PwnedLabs access restriction.
- Architecture Decisions: Group-based access, EC2 role trust, and bucket-scoped permissions.
- Recommendations: Least privilege, credential protection, access reviews, and further validation.
- Threat Model: Assets, threats, potential impacts, and mitigations.
- AI-Assisted Analysis: Policy review supported by ChatGPT and checked against the recorded configuration.
- Appendices: Supporting screenshots and CLI logs.

Key Outcomes

The broad `AmazonS3ReadOnlyAccess` policy was removed from `intern_group` and replaced with `InternLabBucketReadOnly`.

Policy simulation confirmed:

| Test | Result |
|---|---|
| List the permitted lab bucket | Allowed |
| Read an object inside the lab bucket | Allowed |
| Read an object outside the lab bucket | Implicit deny |
| Upload an object inside the lab bucket | Implicit deny |
| Delete an object inside the lab bucket | Implicit deny |

These results demonstrate the tested permission boundaries. They do not establish live access under every applicable AWS control.

## Guided Lab Status

- **Cybr:** IAM Enumeration lab completed, as recorded in Phase E.
- **PwnedLabs:** IAM enumeration, role assumption, and permitted secret retrieval performed. S3 flag retrieval remained blocked by `AWSCompromisedKeyQuarantineV3`.

The PwnedLabs lab is not recorded as fully completed because the flag was not retrieved or submitted.

 Final Deliverable

The report will be uploaded to this folder as:

`AWS_IAM_Security_Report.pdf`

 Completion Checklist

- [ ] Final PDF uploaded
- [ ] Executive Summary, Methodology, Findings & Risks, and Recommendations included
- [ ] Architecture decisions and risk mitigations explained
- [ ] Threat model diagram included
- [ ] AI-assisted analysis summary included
- [ ] Screenshots and logs attached as appendices
- [ ] Sensitive credentials and secret values excluded

Lessons Learned

This task strengthened my ability to turn technical evidence into a structured security report. I learned to distinguish simulated results from live access, explain how policy changes reduce risk, and document incomplete lab outcomes accurately.


