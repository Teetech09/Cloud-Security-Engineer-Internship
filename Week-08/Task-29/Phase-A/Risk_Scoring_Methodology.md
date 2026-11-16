Risk Scoring Methodology

1. Purpose

This document defines the qualitative risk-rating methodology used to evaluate AWS security findings identified during the flaws.cloud assessment.

The methodology combines likelihood and impact to determine the priority of each finding.

2. Likelihood Criteria

| Rating | Description |
|---|---|
| Low (1) | Exploitation requires significant prerequisites or unlikely conditions. |
| Medium (2) | Exploitation is possible under identifiable conditions. |
| High (3) | Exploitation is straightforward or requires minimal prerequisites. |

3. Impact Criteria

| Rating | Description |
|---|---|
| Low (1) | Limited effect on information, resources, or services. |
| Medium (2) | Moderate exposure, modification, or disruption affecting identifiable resources. |
| High (3) | Significant exposure of sensitive information, unauthorized control, or major service disruption. |

4. Risk Calculation

Risk Score = Likelihood × Impact

5. Risk Rating Matrix

| Likelihood / Impact | Low (1) | Medium (2) | High (3) |
|---|---:|---:|---:|
| Low (1) | 1 - Low | 2 - Low | 3 - Medium |
| Medium (2) | 2 - Low | 4 - Medium | 6 - High |
| High (3) | 3 - Medium | 6 - High | 9 - Critical |

6. Risk Classification

| Score | Risk Level | Recommended Priority |
|---|---|---|
| 1–2 | Low | Routine remediation |
| 3–4 | Medium | Planned remediation |
| 6 | High | Prioritised remediation |
| 9 | Critical | Immediate attention |

7. CIA Impact Assessment

Each finding must also be evaluated against the CIA triad:

- Confidentiality: Could an unauthorized party access sensitive information?
- Integrity: Could an unauthorized party modify data, configurations, or resources?
- Availability: Could the weakness cause service disruption or prevent legitimate access?

CIA assessments provide additional context for understanding the consequences of each finding.

8. Application Guidelines

1. Base likelihood and impact ratings on observed evidence and realistic attack conditions.
2. Consider the sensitivity and business importance of affected resources.
3. Avoid assigning a critical rating solely because a particular AWS permission appears dangerous.
4. Document assumptions and limitations supporting each rating.
5. Recommend remediation proportionate to the identified risk.
6. Reassess the risk after remediation has been validated.

This is a qualitative assessment model developed for this learning project. It is not an official AWS risk-scoring standard.
