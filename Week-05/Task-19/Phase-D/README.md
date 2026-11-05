Phase D: Independent Extension & Threat Modeling

 Objective
 
The objective of this phase was to extend the Lambda environment with an SQS-triggered function and to perform a STRIDE threat model of the S3 and SQS event pipeline, ranking each threat in a risk matrix by likelihood and impact.

Actions Taken

I created an SQS queue named `lambda-threat-model` together with a dead-letter queue, `lambda-threat-model-dlq`, which receives a message after three failed receives. I then created a separate IAM role, `LambdaSQSRole`, for the new function instead of reusing `LambdaExecRole`. It has `AWSLambdaBasicExecutionRole` for logging and an inline policy allowing only `sqs:ReceiveMessage`, `sqs:DeleteMessage` and `sqs:GetQueueAttributes` on the one queue ARN. I chose this over the AWS managed SQS execution policy because that policy grants the same actions on all queues.

I deployed a Python function named `SecureSQSProcessor` (python3.12, 5 second timeout) and connected it to the queue with an event source mapping using a batch size of 5 and a maximum concurrency of 2. The function validates every message before using it, checking the size limit, that the body is valid JSON, that it has exactly the expected fields, and that the action is on an allow list. It logs only the message ID and outcome, and it never acts on message content.

I tested the function with three messages: one valid, one that was not valid JSON, and one with a disallowed action. I then built a STRIDE threat table covering the six threat categories and a three-by-three risk matrix ranking each threat by likelihood and impact.

 Key Observations
The valid message was logged as `ACCEPTED` and the other two were logged as `REJECTED`, one for invalid JSON and one for failing schema validation. This shows that input validation handles the Tampering threat, because untrusted queue content is checked before it is used. The event source mapping reported `Enabled` with the expected batch size and concurrency limit, and the queue attributes confirmed the dead-letter redrive policy of three receives and a visibility timeout of 30 seconds.

The threat model produced three High risks and three Medium risks:

| Rank | Threat | Likelihood | Impact | Risk |
|---|---|---|---|---|
| 1 | Spoofing: compromised IAM role | Medium | High | High |
| 2 | Information Disclosure: overly broad permissions | Medium | High | High |
| 3 | Tampering: malicious payloads in events | High | Medium | High |
| 4 | Repudiation: missing audit logs | Medium | Medium | Medium |
| 5 | Denial of Service: excessive invocations | Medium | Medium | Medium |
| 6 | Elevation of Privilege: cross-service trust abuse | Low | High | Medium |

The three High risks share a theme: what an attacker can influence in an event, or steal from a function, is limited only by the execution role. Phase C supports this directly, because adding `s3:*` on `*` let the same function list every bucket in the account and delete an object. The main residual risk is Repudiation, because S3 object-level actions are not recorded in CloudTrail unless data events are enabled.

The controls built into this phase map to the table: a separate least-privilege role per function (Spoofing, Information Disclosure), input validation and a dead-letter queue (Tampering), a maximum concurrency limit and the dead-letter queue (Denial of Service), and `SourceArn` and `SourceAccount` conditions on the S3 invoke permission (Elevation of Privilege).

 Evidence Produced
* PhaseD_threat_model.pdf`: STRIDE threat table with a likelihood and impact rating and a mitigation for each threat, plus the risk matrix, ranking and key findings.
* PhaseD_sqs_setup.png: Lambda console showing `SecureSQSProcessor` with the SQS trigger attached.
* PhaseD_sqs_log.png: CloudWatch Logs output showing one accepted and two rejected messages.

![SQS trigger configuration](phaseD_sqs_setup.png)

![SQS function test log](phaseD_sqs_log.png)
