Phase A: Theory & Introduction

Objective
The objective of this phase was to understand how AWS Lambda executes code securely using temporary credentials derived from IAM roles, and how permissions flow from the invoking service to the function's execution role.

Actions Taken
I reviewed the execution role section of the AWS Lambda Developer Guide to understand how each function is tied to an IAM role that defines what the function can do on AWS. I then explored how event sources such as Amazon S3 and API Gateway invoke Lambda functions, and studied how permissions flow in two separate steps: first when the event source invokes the function, and then when the function calls other AWS services. Finally, I summarized the difference between resource-based and identity-based policies and built a diagram of the full permission flow, saved as `phaseA_summary.png`.

Key Observations
Lambda uses two independent permission checks. The first is invocation permission. When an event source such as S3 or API Gateway triggers a function, Lambda checks the function's resource-based policy, which must allow the calling service (for example `s3.amazonaws.com`) to perform `lambda:InvokeFunction`. This can be narrowed with `SourceArn` and `SourceAccount` so that only a specific bucket or API can trigger the function.

The second is execution permission. Once the function runs, Lambda assumes the execution role through AWS STS, whose trust policy allows `lambda.amazonaws.com`. STS issues temporary credentials that are injected into the runtime environment as `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` and `AWS_SESSION_TOKEN`. The function code uses these credentials to call services such as DynamoDB, S3 and CloudWatch Logs, and the role's identity-based policies decide which calls are allowed.

This design is secure because no long-term access keys are stored in the code or configuration, the credentials expire and rotate automatically, and invocation rights are separated from execution rights. Applying least privilege to both the resource policy and the execution role limits the impact if a function is misused or compromised.

The difference between the two policy types is where they attach and what they answer. An identity-based policy is attached to an IAM user, group or role and defines what that identity can do. A resource-based policy is attached to the resource, such as a Lambda function, S3 bucket or SQS queue, and defines who can access it, so it must name a principal. Within one account, an allow from either type is enough, while cross-account access requires both sides to allow it.

Evidence Produced
phaseA_summary.png is a diagram of the Lambda permission flow showing the event source, the Lambda function, the resource-based policy, the execution role, STS temporary credentials and the downstream AWS services.

![Lambda permission flow](phaseA_summary.png)
