Phase C – Responsibility Mapping

Objective

The objective of this phase was to map AWS and customer responsibilities for IAM, S3, EC2, and CloudTrail, identify potential security risks, highlight at least one misconfiguration, and visualize a potential attack path.

Responsibility Mapping

Service        | AWS Responsibilities                       | Customer Responsibilities     |Potential Risk

IAM            | Secure the IAM service and 
                underlying AWS infrastructure.          | Manage users, roles, permissions,  |Excessive permissions or
                                                          MFA, and least-privilege access.   Weak authentication could                                                                                                    result in unauthorized AWS                                                                                                   access.
S3             | Secure the underlying storage          | Configure bucket policies, Block   | Misconfigured bucket
                infrastructure and S3 service.            Public Access, encryption, and      permissions could expose 
                                                          access permissions.                 sensitive data or CloudTrail      
                                                               
EC2            | Secure the physical infrastructure,    | Secure and patch the guest OS,     | Public exposure, weak                         networking, and virtualization layer.    Configure security groups, manage     security-group rules, or                                                              SSH access, and secure applications.   Unpatched software could
                                                                                               lead to unauthorized access.
                                                       
CloudTrail     | Provide and maintain CloudTrail        | Enable trails, configure           |  Inadequate logging or                        logging service and infrastructure.      logging, select the S3 destination,    unauthorized access,                                                                   and protect log files.                to logs could reduce                                                                                                         security visibility.
                                                                                                                             
                              
Misconfiguration Identified

EC2 Security Group – Unrestricted Outbound Traffic

Observation:
The EC2 security group's outbound rule allows:

- Protocol: All
- Port range: All
- Destination: "0.0.0.0/0"

Potential Risk:
If the EC2 instance is compromised, unrestricted outbound traffic could allow it to communicate with any IPv4 destination.

Recommended Mitigation:
Review the application's required outbound connections and restrict outbound traffic to necessary destinations and ports where operationally feasible.

Risk Level: Medium

Other Security Observations

EC2 Public IPv4 Address

The EC2 instance has a public IPv4 address, which increases its network exposure.

However, SSH access through TCP port 22 is restricted to a single "/32" source IP rather than being open to all internet addresses.

S3 CloudTrail Log Bucket

The S3 bucket has Block Public Access enabled, and its bucket policy restricts access to the CloudTrail service. Public exposure was therefore not observed during this assessment.

EC2 IAM Role

No IAM role is currently attached to the EC2 instance. Therefore, an overprivileged EC2 IAM role was not observed.

Risk Matrix

Threat| Impact| Likelihood| Mitigation
S3 CloudTrail log exposure| High| Low| Maintain Block Public Access and restrictive bucket policy.
Unauthorized access to CloudTrail logs| High| Low| Apply least privilege and monitor access.
Unauthorized SSH access| High| Low| Restrict SSH to trusted sources and maintain secure authentication.
EC2 public network exposure| High| Medium| Minimize exposed services and restrict inbound traffic.
Unrestricted EC2 outbound traffic| Medium| Medium| Review and restrict outbound destinations where feasible.

Attack Path

Potential attack path identified:

Internet → EC2 Public IP → Allowed Network Service → EC2 Instance

SSH access is currently restricted to a single source IP, reducing exposure of TCP port 22.

CloudTrail logging follows a separate path:

AWS CloudTrail → S3 CloudTrail Log Bucket

The S3 bucket uses Block Public Access and a restrictive bucket policy to protect the logs.

Key Findings

- AWS manages the underlying infrastructure for IAM, S3, EC2, and CloudTrail.
- The customer is responsible for securely configuring these services.
- EC2 has a public IPv4 address.
- EC2 SSH access is restricted to a single "/32" source IP.
- EC2 outbound traffic currently allows all IPv4 destinations.
- S3 Block Public Access is enabled.
- No IAM role is attached to the EC2 instance.

Lessons Learned

This exercise helped me understand how the AWS Shared Responsibility Model applies to different cloud services and how customer-side configuration can introduce security risks.

I also learned how to identify configuration risks, assess their potential impact, and document appropriate mitigations.

Evidence

Evidence collected for this phase includes:

- IAM configuration
- S3 Block Public Access configuration
- S3 bucket policy
- EC2 instance configuration
- EC2 security-group rules
- CloudTrail configuration
- Risk matrix
- Attack-path diagram

Note: Sensitive information such as AWS account IDs and IP addresses should be redacted before publishing screenshots publicly.
