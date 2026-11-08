Phase F: Documentation and Reporting

 Objective

Consolidate all findings from Phases A to E into a professional risk analysis suitable for executive review. This report summarizes the actions taken in each phase and the key observation that emerged from the work.



Actions Taken (Phases A to E)

Phase A: Theory and Introduction

- Studied the OWASP Serverless Top 10 and summarized the main risks for serverless functions, with attention to how Lambda functions get their permissions and where secrets and injection weaknesses appear.
- Recorded the summary as evidence (`phaseA_summary.png`) to set the framework for the later phases.

 Phase B: Environment Setup

- Deployed the CloudGoat `lambda_privesc` scenario in WSL2 Ubuntu using the dedicated `cloudgoat` AWS profile.
- Reviewed what it created: only IAM resources, namely the user `chris`, a `lambdaManager` role and an admin debug role.
- Confirmed the account's Lambda functions with `aws lambda list-functions` and captured them as evidence (`phaseB_lambda_list.png`).

Phase C: Enumeration and Analysis

- Installed CloudFox 2.0.5 from the official GitHub release and ran `cloudfox aws lambdas` and `cloudfox aws env-vars` under the `cloudgoat` profile.
- Found two functions and no environment variables in the lab account.
- Inspected the roles with `aws iam get-role` and checked for `AdministratorAccess` and wildcard actions. The debug role had `AdministratorAccess`.
- Saved redacted evidence (`phaseC_roles.json`, `phaseC_envvars.json`, `phaseC_findings.png`).

Phase D: Independent Extension and Threat Modeling

- Built a risk matrix of the Lambda assets against their threats, with eight risks (R1 to R8) scored on a 5x5 heat map.
- Drew a threat model diagram and wrote risk statements with least-privilege mitigations.
- Saved the results as `phaseD_risk_matrix.xlsx` and `phaseD_threat_model.png`.

Phase E: Hosted Labs Integration

- Lab 1, Introduction to AWS Lambda Enumeration:
  - Listed the functions and audited the environment variables of all of them.
  - Downloaded the `payment-processor` code through the temporary URL from `get-function`.
  - Searched the code with `grep` and found a hardcoded secret key and webhook secret, both exposed through logging and debug output.
- **Lab 2, Pentesting AWS Lambda with CloudFox:**
  - Installed CloudFox in CloudShell and confirmed the lab identity with `aws sts get-caller-identity`.
  - Enumerated four functions with `cloudfox aws lambdas`, and found that all used a scoped role and none was admin.
  - Read the environment variables of `database-connector`, which held plaintext credentials.
  - Reviewed the `file-processor` code, which was clean.
  - Invoked `public-data-api` with `{"operation":"admin"}`, and it returned administrative data without authentication.
- Captured completion screenshots (`PhaseE_lab1_completion.png`, `PhaseE_lab2_completion.png`).


Key Observation

Across all five phases, the biggest Lambda risks came from how functions are configured and who can reach them, not from the function runtime itself. The same pattern appeared at every level. In my own CloudGoat environment the problem was identity: an admin-level debug role could turn a small foothold into full control of the account. In the hosted labs the roles were correctly scoped, yet the functions leaked secrets and privileged data through plaintext environment variables, hardcoded code, and an unauthenticated admin path.

Reading permissions turned out to be as dangerous as running code. Access to read a function's configuration or download its package was enough to collect access keys, database credentials, a JWT secret and a payment provider's secret key. That means "read-only" access in a Lambda account has to be treated as sensitive, and the people and services allowed to read function details should be limited as strictly as those allowed to invoke them.

The two environments also showed why one clean result is not enough. My own account had no environment-variable secrets and still contained an over-privileged role, while the lab accounts had properly scoped roles and badly exposed secrets. Each check found something the others missed, so a complete review has to cover roles, configuration, source code, and the logic that decides who may call each function.

Tooling made this practical. CloudFox gave a fast inventory of functions, regions and roles, and the AWS CLI let me examine suspicious functions in detail. Together they show a workable order for a real assessment: inventory first, then permissions, then configuration and code, then behavior testing.

The risk analysis in Phase D ties it together. The recommended controls remain consistent with what the labs showed: move secrets to Secrets Manager or Parameter Store, enforce authorization inside functions, keep execution roles at least privilege (and remove admin roles that aren't needed), and tightly restrict permissions to read, download and invoke functions.
