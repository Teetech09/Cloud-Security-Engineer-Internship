Phase A: Theory & Introduction

Objective

Understand how attackers exploit weak credentials and misconfigured IAM policies to gain initial access to cloud environments, and how the Principle of Least Privilege helps mitigate these risks.

Actions Taken

1. Reviewed the MITRE ATT&CK Cloud framework to understand Initial Access techniques used by attackers to gain entry into cloud environments.
2. Studied how attackers exploit leaked AWS access keys, weak passwords, password spraying, and compromised credentials.
3. Examined examples of overly permissive AWS IAM policies, particularly policies using `"Action": "*"` and `"Resource": "*"`.
4. Used ChatGPT to generate a summary explaining how excessive IAM permissions increase security risks and how the Principle of Least Privilege helps reduce the potential impact of compromised credentials.
5. Saved the AI-generated summary as `phaseA_summary.png` for evidence documentation.

Key Observations

- Compromised credentials can provide attackers with legitimate access to cloud environments, making unauthorised activities harder to distinguish from normal user operations.
- Exposed AWS access keys can allow attackers to authenticate programmatically as the associated IAM identity.
- Weak passwords increase the likelihood of account compromise through password guessing and password spraying.
- Overly permissive IAM policies can expose cloud resources to unauthorised access, modification, or deletion.
- The use of `"Action": "*"` and `"Resource": "*"` can grant unnecessarily broad permissions when not justified by operational requirements.
- The Principle of Least Privilege reduces the potential impact of compromised credentials by limiting identities to only the actions and resources required for their responsibilities.

Security Recommendations

- Enforce strong password policies and multi-factor authentication (MFA).
- Avoid hardcoding or publicly exposing AWS access keys.
- Use temporary credentials and IAM roles wherever possible.
- Apply least-privilege permissions to IAM users, groups, and roles.
- Regularly review IAM policies and remove unnecessary permissions.
- Monitor authentication activities and investigate suspicious access attempts.

Evidence

| Evidence File | Description |
|---|---|
| PhaseA_summary.png | Screenshot of the AI-generated summary explaining Initial Access, weak credentials, over-permissive IAM policies, and least privilege. |

Tools and References

- ChatGPT – AI-assisted theoretical summary.
- MITRE ATT&CK Cloud Matrix.
- AWS Identity and Access Management (IAM) documentation.

Conclusion

This phase established an understanding of how attackers exploit compromised credentials and excessive IAM permissions to gain initial access to cloud environments. It also highlighted the importance of strong authentication and least-privilege access controls in reducing the risk and potential impact of cloud security incidents.
