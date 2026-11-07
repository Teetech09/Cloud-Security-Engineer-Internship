Phase B: Environment Setup (CloudGoat lambda_privesc)

Objective

The goal of this phase was to deploy the CloudGoat `lambda_privesc` scenario so I would have a vulnerable serverless environment to enumerate in the later phases, and to confirm the deployment from the command line by listing Lambda functions.

Actions Taken

I worked from WSL2 Ubuntu on my Windows laptop. My first attempt failed because plain `wsl` opened the Docker Desktop shell instead of Ubuntu, which has no `git`, so I switched to `wsl -d Ubuntu` and carried on from my home folder.

I already had CloudGoat from an earlier lab, and `git pull` reported it was up to date. The brief's `./cloudgoat.py` script no longer exists in this version, because CloudGoat is now installed as a `cloudgoat` command through pipx. The `config profile` step was also replaced by `config aws`. I kept my saved profile (`cloudgoat`), refreshed the IP whitelist with `cloudgoat config whitelist --auto`, and deployed the scenario:

```bash
cloudgoat create lambda_privesc
```

Terraform finished with 9 resources added and none failed. My first `aws lambda list-functions` call failed with a "could not connect to the endpoint" error. I checked DNS and reached the endpoint with `curl`, which returned a 403 (expected for an unsigned request), so the network was fine and the error was a temporary hiccup. After I retried, the command worked. I turned off the AWS CLI pager so the output printed cleanly, then listed the functions:

```bash
export AWS_PAGER=""
aws lambda list-functions --region us-east-1 --profile cloudgoat \
  --query "Functions[].{Name:FunctionName,ARN:FunctionArn,Role:Role}" --output table
```

Key Observations

The scenario did not create a Lambda function of its own. It deployed only IAM resources: a low-privilege user (`chris`) who can assume roles and read IAM, a Lambda manager role with `lambda:*` and `iam:PassRole`, and a debug role with `AdministratorAccess` that Lambda is allowed to assume. The attack path is to create a new function, pass it the admin role, and run it, which ties back to the over-privileged role risk from Phase A.

The two functions shown in the CLI output, `RotateAppDBCredentials` and `RetrieveSecretFunction`, are from my own earlier Secrets Manager work and are not part of CloudGoat. I recorded their ARNs below for my notes.

| Function | ARN |
|---|---|
| RotateAppDBCredentials | `arn:aws:lambda:us-east-1:<account-id>:function:RotateAppDBCredentials` |
| RetrieveSecretFunction | `arn:aws:lambda:us-east-1:<account-id>:function:RetrieveSecretFunction` |

The `create` output printed the access key for `chris`. I kept that out of the screenshot and this repo. It is a disposable lab credential that gets deleted when the scenario is destroyed.

Evidence Produced

PhaseB_lambda_list.png: Terminal output of `aws lambda list-functions` showing the function names, ARNs and roles.

