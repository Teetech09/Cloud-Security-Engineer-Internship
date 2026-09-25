# Phase F — Final Documentation and Reporting

I brought together the work from Phases A–E in `CSE_AWS_T001_Report.pdf`. The report covers the AWS Shared Responsibility Model, the security baseline, responsibility mapping, risk assessment, threat model, guided labs, findings, and recommendations.

## Submission files

- `CSE_AWS_T001_Report.pdf` — final task report
- Evidence from Phases A–E — submitted in the required evidence ZIP

## Main conclusion

The assessment found useful baseline controls, including restricted SSH access, S3 Block Public Access, and multi-region CloudTrail logging. The EC2 security group’s broad outbound IPv4 rule needs review against the instance’s actual requirements. The PwnedLabs exercise separately demonstrated how CloudTrail can reconstruct a role change followed by S3 object access in an authorised lab.
