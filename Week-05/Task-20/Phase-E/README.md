Phase E: Hosted Labs Integration

Objective

Reinforce learning through guided external labs. This phase covers two hands-on Cybr labs focused on enumerating and pentesting AWS Lambda functions.

Evidence

| Lab | Evidence file |
|-----|---------------|
| Introduction to AWS Lambda Enumeration | `PhaseE_lab1_completion.png` |
| Pentesting AWS Lambda with CloudFox | `PhaseE_lab2_completion.png` |

Labs Completed

|  | Lab | Level | Platform | Status |
|---|-----|-------|----------|--------|
| 1 | [Introduction to AWS Lambda Enumeration](https://cybr.com/hands-on-labs/lab/introduction-to-aws-lambda-enumeration/) | Beginner | Cybr | Completed |
| 2 | [Pentesting AWS Lambda with CloudFox](https://cybr.com/hands-on-labs/lab/pentesting-aws-lambda-with-cloudfox/) | Intermediate | Cybr | Completed |



Lab 1: Introduction to AWS Lambda Enumeration

Environment: Cybr's built-in lab terminal (AWS CLI, using the lab's provided credentials).

Scenario: Act as a security analyst auditing the Lambda functions in an account with poor documentation.

 Action Taken and Key Observation

| # | Action Taken | Key Observation |
|---|--------------|-----------------|
| 1 | Listed all deployed functions with `aws lambda list-functions` | Three functions were deployed: `payment-processor`, `api-key-rotator` and `database-connector` |
| 2 | Looped over every function with `aws lambda get-function-configuration` and saved the environment variables to `env_vars_audit.txt` | `api-key-rotator` held an AWS access key pair in plaintext, and `database-connector` held a plaintext database password, host and user |
| 3 | Retrieved the code download location for `payment-processor` with `aws lambda get-function --query 'Code.Location'` | The location is a presigned link that expires after 10 minutes, but it gives full access to the function's source |
| 4 | Downloaded and extracted the deployment package, then searched it with `grep -ri "password\|secret\|key\|token"` | A payment provider secret key and a webhook secret were hardcoded in `index.js`; the key was also logged and returned in a debug response |
| 5 | Submitted the CTF flag and completed the remaining lab steps | Lab marked complete (see `PhaseE_lab1_completion.png`) |

Findings

| Function | Finding | Risk |
|----------|---------|------|
| `payment-processor` | Secret key and webhook secret hardcoded in source; key written to logs and returned in debug output | Credential exposure in source code and logs |
| `api-key-rotator` | AWS access key pair stored in plaintext environment variables | A key rotation function relying on static keys |
| `database-connector` | Plaintext database credentials in environment variables | Database compromise if configuration is readable |

---

## Lab 2: Pentesting AWS Lambda with CloudFox

**Environment:** AWS CloudShell, signed in as the lab's `LambdaSecurityAssessmentUser`. CloudFox v2.0.5 installed from the official release.

**Scenario:** Use CloudFox and the AWS CLI to find misconfigurations and exposed secrets in Lambda functions.

Action Taken and Key Observation

|  | Action Taken | Key Observation |
|---|--------------|-----------------|
| 1 | Confirmed the identity in use with `aws sts get-caller-identity` | Running as `LambdaSecurityAssessmentUser` in the lab account, not my own account |
| 2 | Downloaded, extracted and installed CloudFox, then verified with `cloudfox --help` | CloudFox v2.0.5 ready; the AWS modules include `lambdas`, `env-vars` and `secrets` |
| 3 | Ran `cloudfox aws lambdas` | Four functions in `us-east-1` (`file-processor`, `api-gateway-backend`, `public-data-api`, `database-connector`), all using `ScopedLambdaExecutionRole`, none flagged as an admin role |
| 4 | Read the environment variables of `database-connector` with `aws lambda get-function-configuration` | An access key, a database URL containing a password, a JWT secret, an encryption key and a DB password were all stored in plaintext |
| 5 | Downloaded and reviewed the code of `file-processor` | No sensitive data in the code, so the exposure was not present in every function |
| 6 | Invoked `public-data-api` with the payload `{"operation":"admin"}` using `aws lambda invoke` | The function returned sensitive administrative information with no authentication or authorization check, and the lab flag was in the response |

Findings

| Function | Finding | Risk |
|----------|---------|------|
| `public-data-api` | Returns administrative data to any caller who sends an `admin` operation | Unauthenticated privileged functionality |
| `database-connector` | Multiple credentials in plaintext environment variables | Everything is exposed to anyone who can read the function configuration |
| All four functions | Scoped execution roles; none are admin | Positive control: the exposure came from code and configuration, not an over-privileged role |



Key Observations Across Both Labs

1. **Secrets in the wrong place.** Both labs showed credentials in Lambda code or environment variables, which anyone with read access to the function can see.
2. **Read-only permissions are enough to cause damage.** `lambda:GetFunction` and `lambda:GetFunctionConfiguration` alone exposed source code and credentials.
3. **Authorization must live inside the function.** A payload field such as `operation` is not an access control.
4. **Scoped roles do not fix insecure code.** CloudFox confirmed none of the roles were admin, yet the functions still leaked sensitive data.

Recommendations

1. Move secrets out of code and environment variables. Use AWS Secrets Manager or SSM Parameter Store and fetch secrets at runtime through the function's IAM role.
2. Add authorization inside the function. Validate caller identity and permissions before running privileged actions.
3. Restrict who can invoke and read functions.Limit `lambda:InvokeFunction`, `lambda:GetFunction` and `lambda:GetFunctionConfiguration` to the principals that need them.
4. Use IAM roles instead of static access keys inside Lambda functions, and rotate or revoke any key that has been exposed.
5. Do not log secrets, and remove debug code paths that return them.
6. Keep least-privilege execution roles and audit them regularly with tools such as CloudFox.

 Challenges and How I Solved Them

- Wrong lab opened: one launch landed on a different Cybr item (a course page). Opening the lab through the direct link while signed in fixed it.
- Missing tools and paste problems in the lab terminal: pasted commands arrived with stray terminal characters, which made tools appear missing. Turning off bracketed paste (`bind 'set enable-bracketed-paste off'`) and typing commands manually resolved it.
- CloudShell lag: pasted lines doubled and commands ran together. Using short one-line commands, saving the payload to a file (`file://p.json`) and waiting for the prompt between commands worked.
- Large CloudFox download and extraction prompts: used `unzip -o` to overwrite without prompting and added the binary folder to `PATH`.

Skills Demonstrated

- AWS Lambda enumeration with the AWS CLI
- Detecting hardcoded and plaintext secrets in code and configuration
- Cloud security tooling (CloudFox) installation and use
- Testing for missing authorization on a function
- Troubleshooting cloud terminals and writing remediation guidance

 Notes

- All credentials and secrets seen in these labs belong to temporary lab environments and are intentionally not reproduced in this repository.
- Lab flags are omitted to avoid spoilers.
