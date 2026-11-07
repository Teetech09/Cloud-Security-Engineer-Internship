Week 5 – Task 20: Serverless Function Enumeration

Phase A: Theory & Conceptual Introduction

1. Objective

The objective of this phase was to understand how serverless functions operate within Amazon Web Services (AWS), particularly AWS Lambda, and why enumeration is important during cloud security assessments.

Serverless computing allows developers to execute application code without managing the underlying server infrastructure. Although AWS manages the infrastructure, security configurations, permissions and application code remain important responsibilities.

2. Understanding AWS Lambda Architecture

AWS Lambda is an event-driven, serverless computing service that executes code in response to specific events.

Its major components include:

- Lambda Functions: Application code executed by AWS Lambda.
- Triggers: Events that initiate function execution, such as S3 uploads, API Gateway requests and EventBridge events.
- Execution Roles: IAM roles that determine which AWS resources and services a Lambda function can access.
- Environment Variables: Configuration values made available to functions during execution.
- Amazon CloudWatch: Provides logging and monitoring capabilities for Lambda functions.

### 3. Common Lambda Misconfigurations

Three common security misconfigurations identified during this review include:

1. Over-Privileged IAM Execution Roles**

Lambda functions may be assigned excessive permissions, allowing them to access resources beyond their operational requirements. For example, a function that only needs to read files from one S3 bucket may have unnecessary access to multiple buckets.

2. Exposed Secrets in Environment Variables**

Sensitive information such as API keys, database credentials and access tokens may be stored in environment variables without appropriate protection or secret-management practices.

3. Insecure Function Configurations**

Incorrect invocation permissions, weak input validation and insecure service integrations may expose Lambda functions to unauthorized access or exploitation.

4. OWASP Serverless Top 10 Security Risks

The OWASP Serverless Top 10 (2017 interpretation) highlights ten important application security risks relevant to serverless environments.

| Risk | Description |
|---|---|
| S1 – Injection | Malicious input manipulates application queries or commands. |
| S2 – Broken Authentication | Weak authentication mechanisms allow unauthorized access. |
| S3 – Sensitive Data Exposure | Confidential information is improperly exposed. |
| S4 – XML External Entities (XXE) | Insecure XML processing exposes internal resources or data. |
| S5 – Broken Access Control | Users or functions access resources without proper authorization. |
| S6 – Security Misconfiguration | Incorrect configurations introduce security weaknesses. |
| S7 – Cross-Site Scripting (XSS) | Malicious scripts execute through vulnerable application interfaces. |
| S8 – Insecure Deserialization | Untrusted serialized data is processed unsafely. |
| S9 – Vulnerable Components | Outdated or vulnerable third-party dependencies introduce security risks. |
| S10 – Insufficient Logging and Monitoring | Inadequate monitoring makes security incidents difficult to detect. |

5. Importance of Enumeration in AWS Lambda Security

Enumeration involves identifying and gathering information about resources within an AWS environment.

During a Lambda security assessment, enumeration helps identify:

- Available Lambda functions and their configurations.
- Function runtimes and associated triggers.
- IAM execution roles and attached permission policies.
- Environment variables and potential secret-management weaknesses.
- Lambda layers and third-party dependencies.
- Function invocation permissions.
- Logging and monitoring configurations.

This information helps security assessors identify misconfigurations, excessive permissions and potential attack paths.

6. How IAM Roles Link to Lambda Execution

An IAM execution role defines the AWS permissions available to a Lambda function during execution.

When a function runs, AWS Lambda assumes its assigned execution role and obtains temporary security credentials to perform authorized actions.

For example, a Lambda function responsible for processing uploaded files may require permission to read objects from a particular S3 bucket.

However, it should not automatically receive unrestricted access to every S3 bucket in the AWS account.

This demonstrates the **Principle of Least Privilege**, which requires granting only the permissions necessary to perform an assigned task.

7. Security Recommendations

The following security practices help reduce risks associated with AWS Lambda deployments:

1. Apply least-privilege IAM execution policies.
2. Store sensitive credentials securely using AWS Secrets Manager.
3. Review Lambda resource-based policies and restrict unauthorized invocation.
4. Validate and sanitize incoming event data.
5. Regularly scan and update third-party dependencies.
6. Enable CloudWatch logging and appropriate security monitoring.
7. Regularly review Lambda configurations and associated IAM permissions.

### 8. Phase A Completion Checklist

- [x] Reviewed AWS Lambda architecture, functions and triggers.
- [x] Studied IAM execution roles and their relationship with Lambda.
- [x] Identified three common Lambda deployment misconfigurations.
- [x] Reviewed the OWASP Serverless Top 10 security risks.
- [x] Examined the security implications of exposed secrets.
- [x] Understood the importance of enumeration in cloud security assessments.

9. Evidence

| Evidence | Description |
|---|---|
| `phaseA_summary.png` | Screenshot of the AI-generated theory and conceptual summary. |

Evidence Location:evidence/PhaseA_summary.png, PhaseA_summary2.png

 10. References

1. [OWASP Serverless Top 10](https://owasp.org/projects/serverless-top-10)
2. [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/)
3. [AWS Lambda Execution Role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
4. [AWS Lambda Environment Variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
5. [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/)

