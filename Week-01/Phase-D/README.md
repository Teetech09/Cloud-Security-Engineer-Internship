# Phase D – Independent Extension: Threat Modeling

## Objective

The objective of this phase was to develop a basic threat model showing how cloud misconfigurations could potentially lead to compromise.

## Business Assets

The main cloud assets considered were:

- Amazon S3 CloudTrail log bucket
- Amazon EC2 instance
- IAM identities and permissions
- CloudTrail security logs

## Threats Identified

The following threats were considered:

1. Unauthorized access to CloudTrail logs
2. Unauthorized SSH access to EC2
3. EC2 public network exposure
4. Unrestricted EC2 outbound traffic
5. Excessive IAM permissions
6. Insufficient protection of security logs

## Risk Assessment

A risk matrix was created to assess each identified threat based on:

- Impact
- Likelihood
- Mitigation

See `risk_matrix.xlsx`.

## Key Misconfiguration

The primary configuration risk identified was unrestricted EC2 outbound traffic.

The security group allows:

- Protocol: All
- Ports: All
- Destination: `0.0.0.0/0`

If the EC2 instance were compromised, this configuration could allow communication with external IPv4 destinations.

## Attack Path

A potential attack path was modeled as:

External Attacker
→ Internet
→ EC2 Public IPv4
→ Security Group
→ EC2 Instance
→ Potential Compromise

A secondary path shows the effect of unrestricted outbound traffic:

EC2 Instance
→ Unrestricted Outbound Traffic
→ External Destinations

The diagram represents a potential attack scenario and does not indicate that the EC2 instance has actually been compromised.

## Mitigation

Recommended controls include:

- Restrict inbound access to required sources.
- Keep SSH access limited to trusted sources.
- Review and restrict outbound traffic where operationally feasible.
- Keep the EC2 operating system patched.
- Apply least-privilege IAM permissions.
- Protect CloudTrail logs using appropriate S3 access controls.
- Monitor cloud activity using CloudTrail.

## Evidence

### Risk Matrix

`risk_matrix.xlsx`

### Threat Model Diagram

`threat_model_diagram.png`

## Lessons Learned

This exercise demonstrated how cloud configuration weaknesses can contribute to security risks and how threat modeling can be used to visualize potential attack paths.

I learned how to connect AWS configuration findings to potential threats, assess risk, and identify appropriate mitigations.
