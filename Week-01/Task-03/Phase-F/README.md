Phase F — Documentation and Reporting

Objective
Compile the AWS CLI lab activities, evidence, findings, and lessons learned into a clear report.

Actions Taken
I compiled documentation for Phases A–E, covering identity verification, AWS CLI profile configuration, least privilege testing, the temporary-session objective, and the Cybr guided lab.

I organised the report into objectives, environment details, tools used, actions taken, observations, findings, defensive controls, recommendations, and reflections. I reviewed the available evidence and distinguished verified results from activities that still required confirmatiKey Observations
Identity verification confirms who is making an AWS request, while permission tests establish which actions that identity can perform. Separate CLI profiles organise credentials and configuration, but IAM policies enforce access restrictions.

Evidence must support each reported result. Lab completion confirms participation in the guided exercises, while individual command outputs demonstrate specific technical outcomes.

Report and Evidence
The final report and sanitised supporting evidence will be linked here after upload.

Evidence includes:
- Phase A identity verification: `phaseA_identity.png`
- Phase B profile configuration: `phaseB_profiles.png`
- Phase C allowed and denied action outputs, where retained
- Phase D MFA session evidence PhaseD_mfa_token
- Phase E Cybr lab completion confirmation

Documentation Limitations
The original CLI authentication-error screenshot was not retained. This experience is recorded as troubleshooting rather than an evidence-backed security finding.


Lessons Learned
- Verify the active identity before running AWS commands.
- Test both required access and intended restrictions.
- Repeat permission tests after IAM policy changes.
- Capture evidence during each activity.
- Exclude access keys, secret keys, and session tokens from reports and GitHub uploads.
