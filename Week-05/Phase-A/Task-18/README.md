 Phase A: Theory & Threat Modeling

Objective

This phase aimed to understand how credentials on an EC2 instance can be compromised through an exposed instance metadata service or a misconfigured IAM role, and to capture that understanding in a threat model, a risk matrix, and a comparison of IMDSv1 and IMDSv2.

Actions Taken

I began by reviewing how the Instance Metadata Service works. Every EC2 instance can reach this service at the link-local address `169.254.169.254`. When an IAM role is attached to the instance, the service exposes that role's temporary credentials at `/latest/meta-data/iam/security-credentials/`, followed by the role name. Under IMDSv1, a single unauthenticated HTTP GET request is enough to retrieve the `AccessKeyId`, `SecretAccessKey`, `Token` and `Expiration` values. This is the reason a web application with a server-side request forgery (SSRF) flaw can be tricked into fetching these credentials on an attacker's behalf.

Next, I studied IMDSv2 and compared it with IMDSv1. IMDSv2 is session-oriented. The caller must first send a PUT request to `/latest/api/token` with a TTL header to obtain a session token, and must then present that token in a header on every metadata request. The token response has a default hop limit of one, and requests carrying an `X-Forwarded-For` header are rejected, so the token cannot easily be relayed through a proxy. The comparison is summarised in the table below.

| | IMDSv1 | IMDSv2 |
|---|---|---|
| Model | Plain request/response | Session-oriented, token required |
| How it is called | One GET to `169.254.169.254` | PUT to `/latest/api/token`, then GET with the token header |
| Credential theft via SSRF | Easy, a simple GET is enough | Much harder, SSRF rarely allows a PUT with custom headers |
| Request forwarding | Nothing prevents proxied requests | Hop limit of 1 by default, `X-Forwarded-For` requests rejected |
| Token lifetime | None | 1 second to 6 hours, set by the caller |
| Enforced by | Default if not changed | Setting `HttpTokens` to `required` on the instance |

I then built the threat model shown in `phaseA_threatmodel.png`. The diagram has two rows. The top row is the attack path, running from an external attacker to an EC2 instance with an SSRF-prone application, then to the IMDSv1 endpoint, and finally to stolen role credentials. The bottom row places one mitigation beneath each link, with a dashed arrow showing which link that control breaks. The asset is the EC2 instance and its IAM role, the threat is credential theft, the vulnerability is IMDSv1 exposure combined with an application flaw, and the mitigations are WAF rules, fixing the application flaw, enforcing IMDSv2 and applying least privilege to the role.

Finally, I completed a risk matrix using a likelihood and impact scale of 1 to 5 and recorded the expected residual score after each mitigation is applied.

| Threat | Vulnerability | L | I | Score | Rating | Mitigation | Residual |
|---|---|---|---|---|---|---|---|
| Credential theft via SSRF | IMDSv1 enabled | 4 | 5 | 20 | Critical | Enforce IMDSv2 (`HttpTokens=required`) | 4 (Low) |
| Abuse of an over-permissive instance role | Misconfigured IAM role | 3 | 5 | 15 | High | Least-privilege policy scoped to specific resources | 6 (Medium) |
| SSRF in the web application | Unvalidated user-supplied URLs | 4 | 4 | 16 | High | Input validation and WAF rules | 6 (Medium) |
| Stolen keys used undetected | No monitoring of role activity | 3 | 4 | 12 | Medium | CloudTrail and GuardDuty alerts | 4 (Low) |

Key Observations

The most important observation is that the metadata service turns a single application flaw into a cloud-level compromise. The attacker never needs to touch AWS directly, because the instance hands its own credentials to anything running on it that can make an HTTP request. This is how the 2019 Capital One breach worked, and it explains why the credential theft threat scored the highest risk in the matrix.

The comparison also showed that IMDSv2 does not remove the SSRF flaw; it removes what the flaw can reach. A simple SSRF cannot send a PUT request with custom headers, so enforcing IMDSv2 cut the likelihood of this attack sharply, which is why its residual score drops from 20 to 4. The mitigation is a one-line configuration change per instance, which makes it very cheap compared with the risk it removes.

The two remaining mitigations show that defence in depth matters. Even with IMDSv2 enforced, an over-permissive role means any other route to the credentials still causes serious damage, so least privilege lowers the impact side of the score. Fixing the application and adding WAF rules addresses the root cause, and CloudTrail and GuardDuty give visibility if stolen keys are ever used. No single control brings every risk to Low, which is why the matrix keeps Medium residual ratings for the role and application rows.

Evidence Produced

The evidence for this phase is phaseA_threatmodel.png, a screenshot of the threat model diagram. The IMDSv1 and IMDSv2 comparison table and the risk matrix are included in this README.
