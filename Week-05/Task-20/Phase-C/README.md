Phase C: Core Execution (Enumeration & Analysis)

Objective

The goal of this phase was to enumerate the Lambda functions in my AWS account, the IAM roles they run as, their triggers and their environment variables, and to look for risks such as over-privileged roles or hardcoded credentials.

Actions Taken

I worked from WSL2 Ubuntu with the `cloudgoat` AWS CLI profile in `us-east-1`. That profile belongs to my lab admin user (`cloudgoat-lab`), not to the scenario's low-privilege user, because Chris only has IAM read permissions and cannot list Lambda functions. The `lambda_privesc` scenario from Phase B was still deployed.

CloudFox was not installed, so I downloaded the Linux release from the project's GitHub page. The zip extracted into a subfolder, and my first `mv` moved the whole folder into `/usr/local/bin`, so the command was not found. I moved the binary out of that folder and checked its SHA1 hash against the `sha1sum.txt` that came with the release. The two hashes matched, so I removed the leftover folder and confirmed that `cloudfox --help` worked.

I then ran the two CloudFox modules that matter for this phase:

```bash
cloudfox aws lambdas --profile cloudgoat
cloudfox aws env-vars --profile cloudgoat
```

The brief wrote the first command as `cloudfox lambda`, but in version 2.0.5 the module is `cloudfox aws lambdas`. The Lambda module covered 17 regions with no errors. The `env-vars` module reported 17 errors out of 66 tasks, and I did not investigate them further.

Next I pulled the role details with the AWS CLI. For each of the four roles (the two roles used by my existing functions, plus the scenario's debug and Lambda manager roles), I ran `aws iam get-role`, `aws iam list-attached-role-policies` and `aws iam list-role-policies`, then read each policy document with `aws iam get-policy-version` or `aws iam get-role-policy`. I combined the role details into `phaseC_roles.json` using `jq`:

```bash
aws iam get-role --role-name <RoleName> --profile cloudgoat
aws iam list-attached-role-policies --role-name <RoleName> --profile cloudgoat
aws iam list-role-policies --role-name <RoleName> --profile cloudgoat
```

For environment variables, I looped over every function with the command from the brief and saved the results to `phaseC_envvars.json`:

```bash
aws lambda get-function-configuration --function-name <FunctionName> \
  --region us-east-1 --profile cloudgoat --query 'Environment.Variables'
```

Finally I checked triggers with `aws lambda list-event-source-mappings` and the resource policy of each function with `aws lambda get-policy`. Because the JSON files contain my account ID, I made redacted copies that replace it with `<account-id>` before uploading them.

Key Observations

CloudFox found exactly two Lambda functions, `RotateAppDBCredentials` and `RetrieveSecretFunction`. Both are left over from my earlier Secrets Manager work, because `lambda_privesc` does not create any function. CloudFox marked neither of their roles as an admin role, so no function in the account currently runs with administrator access.

Neither function has any environment variables. The CLI returned `null` for both and CloudFox reported none, so there are no hardcoded credentials or tokens. This matches the design from my Secrets Manager task, where the functions read the secret at runtime instead of storing it in their configuration.

The two roles used by my own functions follow least privilege. `LambdaSecretAccessRole` has one inline policy that allows only `secretsmanager:GetSecretValue` on the single `AppDBCredentials` secret. The rotation role allows the four rotation actions on the same secret ARN, plus `GetRandomPassword` on `*`, which AWS does not let you scope to a resource. The rotation role also has the standard CloudWatch Logs policy. `LambdaSecretAccessRole` has no managed policies attached, so as far as this enumeration shows it has no logging permissions.

The risky parts of the account come from the scenario itself. `cg-debug-role` has the AWS-managed `AdministratorAccess` policy (`"Action": "*"`, `"Resource": "*"`) and trusts the Lambda service. `cg-lambdaManager-role` allows `lambda:*` and `iam:PassRole` on every resource. Neither role is dangerous alone, but together they let someone create a new function, pass it the admin role and run code as an administrator. This is the over-privileged role risk from Phase A, and it is the escalation path the scenario is built around. CloudFox did not flag it because no existing function uses the debug role yet.

Neither function has an event source mapping, so nothing triggers them automatically. `RotateAppDBCredentials` has a resource policy that lets `secretsmanager.amazonaws.com` invoke it, which is what rotation needs. That statement has no `SourceArn` or `SourceAccount` condition, so I rate it as a low to medium risk (a confused-deputy gap), and adding a condition scoped to my secret would close it. `RetrieveSecretFunction` has no resource policy at all, which means only IAM principals in my account can invoke it.

Evidence Produced

- PhaseC_roles.json: role ARNs, trust policies, and attached or inline policy names for the four roles (redacted copy).
- PhaseC_envvars.json: environment variables for both functions, which are `null` (redacted copy).
- PhaseC_findings.png: terminal screenshot of each role's attached policies, the `AdministratorAccess` wildcard policy, and the empty environment variables.
