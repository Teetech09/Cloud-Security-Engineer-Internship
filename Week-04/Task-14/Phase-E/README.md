Phase E - Hosted Labs Integration

Objective
Complete two Cybr hands-on labs to reinforce learning on Secrets Manager enumeration and secure integration patterns.

Actions Taken

Lab 1: Introduction to Secrets Manager Enumeration
I logged in as the IAM user Julie using AWS CLI credentials provided by the lab, and confirmed my identity with `aws sts get-caller-identity`. I read Julie's inline policy, `AllowReadSecretsManager`, which allowed `secretsmanager:ListSecrets` on all resources, but restricted `GetSecretValue`, `DescribeSecret`, `ListSecretVersionIds` and `GetResourcePolicy` to two specific secrets, `sm-enumerate-password` and `sm-enumerate-api-key`. I retrieved both secret values; one was plain text and the other was base64 encoded. I decoded the encoded value and submitted the resulting flag, which the lab accepted.

Lab 2: Access Secrets Manager via Lambda Function with LFI
I opened the Lambda-powered employee portal and recreated the lab's documented requests. I used the `/create-profile` endpoint to create test employee profiles, then probed the `/find-employee?employee_code=` parameter for injection flaws. A backtick character triggered a `400 Bad Request`, confirming the input wasn't sanitized. In my first session, I successfully exploited the LFI using `employee_code=../../../../../../../../etc/passwd`, which returned the server's `/etc/passwd` content, and then used `employee_code=../../../../../../../../../../proc/self/environ` to read the Lambda function's own process environment, from which I extracted its temporary AWS access key, secret key and session token.

I configured these as a separate AWS CLI profile and attempted to verify and use them against Secrets Manager. [CONFIRM: describe exactly what happened, for example "get-caller-identity did not return a valid assumed-role identity" or "the credentials appeared to expire before I could complete the remaining steps".] After relaunching the lab in a second session to get fresh credentials, I was unable to reproduce the same successful LFI response; repeated attempts at the `/etc/passwd` and `/proc/self/environ` payloads, including varying the traversal depth and URL encoding, returned a generic `{"Message":null}` response instead of file content.

Key Observations
Lab 1 reinforced that `ListSecrets` being broadly allowed while `GetSecretValue` is tightly scoped is a common and sound pattern: a user can see that secrets exist without being able to read most of them, which is the least-privilege model I applied to `LambdaSecretAccessRole` in Phase B. Lab 2 showed a different and arguably more dangerous attack path than enumeration: an application-layer vulnerability, not a credentials leak or an IAM misconfiguration, let an attacker read the Lambda function's own temporary credentials directly from its process environment. Even a correctly scoped IAM role is only as safe as the code calling it, since an LFI flaw hands over whatever permissions that code already has. I was not able to complete lab 2 to the point of retrieving the final flag, despite confirming the vulnerability and stealing valid credentials in an earlier session, which itself was a useful lesson in how short-lived Lambda session credentials are and how quickly a security tester has to move from exploitation to impact before a session expires.

Evidence Produced
- `LabCompletion.png`: completion screenshot for the Secrets Manager Enumeration lab, showing the accepted flag.

Completion Checklist
- Completed both Cybr labs. [NOT FULLY MET: Lab 1 completed; Lab 2 attempted, LFI and credential theft confirmed, but the secret was not retrieved before the lab session expired.]

