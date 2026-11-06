 Phase E: SQS and Lambda SQL Injection (Pwned Labs)

#Objective
Exploit a SQL injection flaw in an AWS serverless workflow where messages sent through SQS are processed by a Lambda function that queries a database.
Understand how untrusted queue input becomes an attack path.

#Environment
- Platform: Pwned Labs
- Services: Amazon SQS, AWS Lambda, [database used]
- Tools: AWS CLI, [other tools]

Actions Taken
1. [Configured AWS CLI with the provided credentials and confirmed identity]
2. [Enumerated SQS queues and Lambda functions]
3. [Reviewed the Lambda function code or configuration]
4. [Crafted and sent a malicious message to the queue]
5. [Retrieved the data or flag]

Key Observations
- [Lambda concatenated message content directly into a SQL query]
- [No input validation or parameterized queries]
- [Permissions were broader than necessary]

 Impact
[What an attacker could read or modify, and why trusting queue messages is dangerous]

#Remediation
- Use parameterized queries or prepared statements
- Validate and sanitize all SQS message content
- Apply least-privilege IAM to the Lambda role and queue policies
- Enable logging and alerting on abnormal queries

 Evidence
 PhaseE_lab_completion.png – Screenshot of lab completion
