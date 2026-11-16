Phase A: Assessment Planning & Cloud Misconfiguration Model

1. Objective

Develop an investigation plan for the flaws.cloud AWS security challenge and understand the trust boundaries, potential misconfigurations, and attack paths associated with the AWS services examined across Levels 1–6.

This phase establishes the assessment scope, rules of engagement, evidence collection approach, and risk assessment methodology before practical testing begins.

2. Action Taken

The following activities were completed as part of the assessment planning:

1. Reviewed the assessment objectives and introductory information for the flaws.cloud challenge.
2. Defined the authorized assessment scope and documented rules of engagement to ensure testing remains within the designated learning environment.
3. Prepared a cloud security assessment plan covering the assessment objectives, scope, methodology, limitations, evidence requirements, and hint usage.
4. Identified the AWS services and trust boundaries relevant to the investigation, including Amazon S3, IAM, STS, EC2, EBS, Instance Metadata Service, Lambda, and API Gateway.
5. Designed a conceptual cloud attack path to illustrate how misconfigurations and exposed credentials could create opportunities for unauthorized access across AWS services.
6. Established the information required for documenting security findings, including misconfiguration details, evidence, attack paths, CIA impact, likelihood, impact, MITRE ATT&CK mapping, remediation, and validation methods.
7. Reviewed the AWS security concepts required for the practical investigation, including least privilege, temporary credentials, public S3 access, snapshot sharing, and EC2 metadata security.

3. Key Observations

A. AWS Trust Boundaries

AWS services operate through permissions, identities, and resource-level access controls. A weakness in one service can potentially expose information or credentials that create further access opportunities in another service.

B. Amazon S3 Exposure

Misconfigured S3 buckets can expose sensitive files, application configurations, or other information to unauthorized users. Public accessibility must be evaluated against the intended business purpose and data sensitivity.

C. IAM Credentials and Permissions

Exposed AWS credentials can provide access to an AWS identity. The actual level of access depends on the identity's effective permissions, applicable resource policies, and other authorization
4. Findings Template and Risk Assessment Methodology

A structured findings template was prepared to ensure that security weaknesses identified during the flaws.cloud assessment can be consistently documented, evaluated, and communicated.

The template captures the affected AWS service, identified misconfiguration, supporting evidence, potential attack path, business impact, risk rating, MITRE ATT&CK mapping, recommended remediation, and validation method.

Findings Template Fields

The following fields were established for documenting assessment findings:

| Field | Description |
|---|---|
| Level | Identifies the flaws.cloud challenge level where the finding was observed. |
| AWS Service | Identifies the affected AWS service, such as S3, IAM, EC2, or EBS. |
| Misconfiguration | Describes the identified security weakness. |
| Evidence | Records screenshots, CLI outputs, configurations, or other supporting information. |
| Attack Path | Explains how the weakness could be exploited and its relationship with other AWS resources. |
| Confidentiality | Evaluates the potential for unauthorized disclosure of information. |
| Integrity | Evaluates the potential for unauthorized modification of data or resources. |
| Availability | Evaluates the potential for service disruption or loss of access. |
| Likelihood | Estimates how likely the weakness is to be exploited. |
| Impact | Evaluates the potential severity of the consequences. |
| MITRE ATT&CK Mapping | Identifies relevant adversary techniques where supported by the observed behaviour. |
| Remediation | Recommends security controls or configuration changes to address the weakness. |
| Validation Method | Defines how to verify that the identified weakness has been corrected. |

Risk Assessment Approach

A qualitative risk-rating approach was established using three likelihood categories and three impact categories:

- Low: Limited likelihood or minimal consequence.
- Medium: Reasonable likelihood or moderate consequence.
- High: High likelihood or significant consequence.

The overall risk rating is determined by combining likelihood and impact using a 3 × 3 risk matrix.

This approach provides a consistent method for prioritising findings and recommending corrective actions.

5. Key Observations – Risk Assessment

- Security findings should be supported by verifiable evidence rather than assumptions.
- The same AWS misconfiguration can have different levels of risk depending on the sensitivity of the affected resources.
- Confidentiality, integrity, and availability must be assessed separately to understand the full potential impact.
- MITRE ATT&CK mapping helps describe adversary behaviour associated with an observed weakness.
- Remediation recommendations should address the underlying configuration weakness rather than only its visible symptoms.
- Validation is necessary to confirm that corrective actions have effectively reduced or eliminated the identified risk.

6. Evidence Produced

- PhaseA_CloudAssessmentPlan.txt — Documents the assessment scope, rules of engagement, methodology, trust boundaries, and evidence collection approach.
- PhaseA_AttackPath.png — Illustrates the conceptual attack path and relationships between AWS services across the challenge levels.
- Findings Template — Defines the fields required to record, assess, and report identified cloud security weaknesses.
- Risk Assessment Methodology — Establishes the likelihood and impact criteria used to prioritise security findings.

7. Phase A Outcome

The assessment planning phase established a structured approach for investigating AWS cloud misconfigurations.

The scope, assessment methodology, evidence requirements, attack-path model, findings structure, and qualitative risk-rating approach were defined before proceeding to practical investigation.

This provides a foundation for identifying security weaknesses, understanding their potential consequences, and recommending appropriate remediation measures.

