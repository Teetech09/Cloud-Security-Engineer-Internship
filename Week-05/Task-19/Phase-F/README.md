Phase F: Documentation and Reporting

Objective
AWS Lambda Security Architecture, Event Models, and Execution Permissions explains how AWS secures serverless functions through their architecture, the events that trigger them, and the IAM roles that define what they can access. This matters for a cloud security engineer because Lambda functions often run with broader permissions than they need and trust event data from sources like S3 and SQS that an attacker may influence, so one weak role or unvalidated input can expose a whole environment. Phases A to E target over-permissive execution roles and injection through event sources, which I explored through theory, a hands-on homelab, threat modeling and a hosted lab simulating a SQL injection attack through an SQS queue.

Actions Taken
I created a folder for each of Phases A to E and wrote a README in each one covering the objective, actions taken, key observations and evidence produced.
I then compiled the full report, including the introduction, MITRE ATT&CK mapping, architecture overview, tool configuration, findings, risk assessment, remediation, reflection questions and AI tool declaration, and saved it as
I named every evidence file by phase and checked each screenshot, log, and policy fragment for access keys, session tokens, account IDs, and lab credentials, redacting anything sensitive before committing. I committed the work with `git add`, `git commit` and `git push origin main`, then opened the repository on GitHub to confirm that the folders, READMEs and images
displayed correctly.

Key Observations
Writing up the earlier phases in my own words showed me where my understanding was thin, and I went back to [the step or concept you revisited] to fix it. Evidence only has value when it is named, captioned and tied to a finding, and
a public repository needs the same review as any production system, because one leaked credential in a screenshot would undo the security lesson the
project teaches. The report structure also showed me how a technical result becomes a finding, a risk and a recommendation that a reader can act on.

Evidence Produced
PhaseE_CSE_AWS_L_019_04_YNT_20261106- Final Report
