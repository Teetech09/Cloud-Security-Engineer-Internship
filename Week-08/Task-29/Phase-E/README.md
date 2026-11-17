Phase E: Defensive Engineering — Hardening & Detection Validation

Objective

Translate the six weaknesses found across flaws. Cloud Levels 1–6 into practical AWS hardening controls, policy validation, and detection requirements that a Cloud Security Engineer could implement in a real environment.

Actions Taken

I built a six-row hardening matrix, one row per challenge level, covering the root cause, the preventive control, the detective control, the responsible AWS service, an implementation priority, and a validation method for each. Priorities were assigned by how severe and how easily exploitable each weakness was in practice: the Level 3 leaked Git credential and the Level 5 SSRF/IMDS credential theft were rated Critical, since both yield a usable, live credential with comparatively low attacker effort; Levels 1, 2, and 4 were rated High, since each exposes data or infrastructure directly but requires a bit more reconnaissance; Level 6 was rated Medium, since it required deliberate, methodical enumeration across three services and yielded discovery of a hidden endpoint rather than direct credential or data exposure.

I then designed hardened policy examples for four challenge themes, one more than the three required: restrictive S3 access (Levels 1–2), credential and secret handling (Levels 3–4), EBS snapshot sharing (Level 4), and, as a bonus, EC2 IMDSv2 enforcement paired with a least-privilege instance role (Level 5). Each theme pairs the vulnerable pattern actually observed in the challenge with a hardened replacement, along with the exact `aws accessanalyzer validate-policy` command needed to validate it in the sandbox account, and notes on IAM Policy Simulator where an identity policy's effective permissions needed confirming rather than a resource policy's syntax.

I documented a detection plan covering five suspicious activities, one more than the four required: public S3 bucket policy or ACL changes, public EBS snapshot or AMI sharing, unusual use of a leaked or long-lived access key, sensitive IAM/Lambda/API Gateway enumeration, and anonymous or unexpected object-level S3 access. For each, I specified whether AWS CloudTrail management events (captured by default, no extra cost) or S3 data events (must be explicitly enabled per bucket/prefix, with added cost) are required, since object-level access to a specific bucket's contents is only visible through data events, while control-plane changes like policy or ACL modifications are management events.

Finally, I built a sandbox review checklist covering the four items specified: S3 Block Public Access status at the account level, the EBS snapshot public-access protection setting in the working region, confirmation that no stale challenge credentials remain configured in local AWS CLI profiles, and verification that every Level 4 temporary resource (instance, volume, security group, snapshot) was deleted. I ran the listed commands against my sandbox account and recorded the results.

Key Observations

Building this matrix after working through the challenge rather than before made the root-cause column far more precise: having actually exploited each weakness firsthand meant the preventive control could be stated specifically (for example, "block Group=all on `ModifySnapshotAttribute`") rather than generically ("secure your snapshots").

Distinguishing management events from data events in the detection plan surfaces a real operational trade-off. Management events are free and always on, which is why four of the five detection use cases here rely on them alone; object-level data events carry a real cost and must be turned on deliberately, so they are worth enabling only on buckets that actually hold sensitive data, not account-wide by default.

Validating the hardened policies rather than just writing them matters because an overly restrictive condition or malformed ARN pattern can silently break intended access just as easily as an overly broad one breaks security; `accessanalyzer validate-policy` and the IAM Policy Simulator are the tools that catch both failure modes before a policy is actually deployed.

The sandbox review step closed the loop on every prior phase: it isn't enough to clean up resources as each level finishes, since account-wide settings like S3 Block Public Access or the EBS snapshot public-access state are posture checks that should be verified independently of any single task, and a credential profile left configured from an earlier level is itself a residual risk worth catching before considering the engagement complete.

Evidence Produced

- PhaseE_HardeningMatrix.xlsx — three-sheet workbook: *Hardening Matrix* (the six-row table covering root cause, preventive control, detective control, responsible AWS service, priority, and validation method for Levels 1–6), *Detection Plan* (five CloudTrail/CloudWatch/EventBridge use cases with management-event vs. data-event distinctions), and *Sandbox Review* (the four required checks with exact commands, expected results, and a status column completed against my sandbox account).
- PhaseE_hardened_policy_examples.json — vulnerable-pattern-to-hardened-replacement pairs for four themes: restrictive S3 access, credential/secret handling via Secrets Manager, EBS snapshot sharing, and EC2 IMDSv2/least-privilege instance roles, each with its corresponding `accessanalyzer validate-policy` validation command.
- PhaseE_PolicyValidation.png — `aws accessanalyzer validate-policy` output for all three hardened policies (S3 bucket policy, Secrets Manager identity policy, least-privilege instance role policy), each returning zero findings.
