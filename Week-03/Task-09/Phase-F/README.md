Phase F: Documentation and Reporting

Objective

Consolidate the configurations, test results, risks, mitigations and lessons learned from Phases A–E into a professional security assessment.

Scope

The task covered:

- Amazon S3 architecture and access-control concepts.
- Private bucket configuration and scoped IAM permissions.
- Authenticated and anonymous access testing.
- SSE-S3 encryption verification.
- Temporary public-read exposure using a separate simulation bucket.
- Two authorized PwnedLabs exercises.

Configuration Summary

| Resource | Configuration |
|---|---|
| Original bucket | student-s3-lab-teetech-20261020 |
| Public access protection | All four Block Public Access settings enabled |
| Object Ownership | Bucket owner enforced; ACLs disabled |
| Versioning | Enabled |
| Default encryption | SSE-S3 |
| IAM user | s3tester; console access disabled |
| IAM permissions | ListBucket, GetObject and PutObject scoped to the original bucket |
| Simulation bucket | student-s3-public-sim-teetech-20261020 |
| Final simulation state | Temporary public policy deleted; all public access blocks restored |

 Findings and Results

| Finding / Observation | Evidence | Security Significance |
|---|---|---|
| Scoped IAM access worked | s3tester successfully uploaded, listed and downloaded the sample object | Required operations were available without granting bucket administration or deletion permissions |
| Anonymous access to the original bucket was denied | Unsigned GetObject requests returned AccessDenied before and after the policy change | The tested object remained inaccessible anonymously |
| Temporary public-read exposure was demonstrated | The simulation object's URL became accessible without authentication | A public policy combined with relaxed protective settings can expose data |
| SSE-S3 did not prevent permitted public reading | The encrypted simulation object was readable during the test | Encryption at rest does not replace access controls |
| Private access was restored | The simulation URL returned AccessDenied after rollback | The temporary exposure was removed |
| Hosted labs demonstrated credential exposure chains | Both PwnedLabs exercises were completed | Public files containing credentials can enable further cloud access |

The original bucket's successful operations demonstrate functional permissions, not exhaustive proof that every out-of-scope action is denied.

Risks and Mitigations

| Risk | Potential Impact | Recommended Mitigation |
|---|---|---|
| Public object-read permissions | Loss of confidentiality if sensitive objects are exposed | Enforce Block Public Access and remove unnecessary public grants |
| Exposed credentials in scripts or archives | Unauthorized access within the credentials' permissions | Remove embedded credentials, rotate exposed keys and use roles with temporary credentials where appropriate |
| Unnecessary access to Parameter Store values | Disclosure of sensitive values and access to additional identities | Scope parameter access precisely and review relevant KMS permissions |
| Reliance on encryption alone | Encrypted objects may still be disclosed through permitted requests | Combine encryption with least privilege and public access restrictions |
| Incomplete visibility into object access | Delayed detection of suspicious reads | Configure appropriate S3 data-event logging or server access logging |

The Phase D likelihood and impact ratings describe potential sensitive-data exposure. The simulation itself used harmless dummy data.

Task Deviations and Limitations

 Public Access Instruction

The Phase B instruction concerning public access overrides was ambiguous. I retained all four Block Public Access settings to meet the private-environment objective.

ACL Requirement

S3 ACLs cannot directly grant permissions to individual IAM users. I retained ACLs disabled and documented this limitation.

No ACL modification or second-user access test was completed.

 Separate Simulation Bucket

I performed the public-read simulation in a separate bucket containing only dummy data. The original bucket's security settings remained unchanged.

Validation Limits

- Authenticated listing and anonymous reading were repeated after the Phase C policy change.
- Authenticated upload and download were not repeated after that change.
- Anonymous access was already denied before the explicit-deny policy was added.
- The denial test did not isolate the effect of each active security control.
- The ffuf scan reported 20,538 errors, so its coverage was incomplete.
- Monitoring recommendations are not claims that those logging controls were configured during this task.

Hosted Labs Completed

1. AWS S3 Enumeration Basics.
2. S3 Bucket Brute Force to Breach.

Completion evidence:

- PhaseE_pwnedlab1_completion.png
- PhaseE_pwnedlab2_completion.png

Both screenshots are stored with the Phase E documentation.

Lessons Learned

S3 resilience, encryption and access control solve different problems and must be assessed separately.

Bucket policies can control individual objects and apply detailed conditions; object ACLs are not required for object-specific access.

A failed browser request does not prove that a bucket is private. Access may depend on identity, object key and request parameters.

Named CLI profiles help separate credentials, but STS verification is needed to confirm the actual identity.

Expected denials are useful evidence of restrictions, while connection failures require separate troubleshooting.

Accurate reporting requires distinguishing completed tests, simulated risks, recommendations and unresolved limitations.

 Evidence Handling

Supporting evidence is organized within the documentation for Phases A–E.

Credentials, secret parameter values, transaction contents and lab flags are excluded from public GitHub evidence.

Conclusion

The original lab bucket supported the required authenticated operations while denying tested anonymous reads. The separate simulation demonstrated that a temporary public-read policy could expose an SSE-S3-encrypted object. Removing the policy and restoring Block Public Access returned the object to a private state.

The hosted labs reinforced the importance of preventing credential exposure, restricting permissions and verifying access through evidence.
