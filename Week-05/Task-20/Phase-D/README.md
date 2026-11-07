Phase D: Independent Extension & Threat Modeling

Objective

The goal of this phase was to build a threat model that shows how misconfigured serverless components could be attacked, rate each risk by likelihood and impact, and propose mitigations that follow the principle of least privilege.

Actions Taken

I used the findings from Phase B (the `lambda_privesc` scenario) and Phase C (my enumeration results) as the source for the model. I listed the assets in scope: the two existing Lambda functions, `RotateAppDBCredentials` and `RetrieveSecretFunction`, plus the user and roles the scenario created (`chris`, `cg-lambdaManager-role` and `cg-debug-role`). For each asset I identified the threats that applied, such as privilege escalation, credential exposure, data leakage and undetected enumeration.

I built the risk matrix in a spreadsheet, `phaseD_risk_matrix.xlsx`, with eight risks. Each one has an attack path, a likelihood and an impact rating from 1 to 5, and a score calculated as likelihood times impact. Ratings follow four bands: Low (1-4), Medium (5-9), High (10-16) and Critical (17-25). A second sheet turns the register into a 5x5 heat map, and a third sheet records the scales and my assumptions.

For the risk statements, I used AI assistance as the brief suggested, with prompts in the style of the example: *"Generate a risk statement for an over-privileged Lambda role allowing lambda:\* and iam:PassRole on all resources."* I then reviewed and edited each statement against my Phase C evidence so that it describes what I actually found. For every risk I wrote a least-privilege mitigation and estimated the residual likelihood and impact after it is applied.

Finally, I drew the attack path and its mitigations as a diagram, `phaseD_threat_model.png`, using the same risk IDs as the spreadsheet.

Key Observations

The model produced eight risks: one Critical, two High, four Medium and one Low. The Critical risk (R1) is the privilege escalation path built into the scenario. The attacker starts as `chris`, assumes the Lambda manager role, creates a new function, passes it the debug role with `iam:PassRole`, and runs it as an administrator. Each step is only a small permission on its own, but the chain ends in full account access. The two High risks are the debug role itself, which carries `AdministratorAccess`, and Chris's long-lived access key, which could let an attacker enumerate IAM and assume roles if it leaked.

The risks around my own two functions rate lower, which fits what I found in Phase C. Both roles are scoped to one secret, neither function stores secrets in environment variables, and nothing triggers them automatically. The remaining concerns are a compromised function reading or overwriting the secret (Medium), a missing `SourceArn` or `SourceAccount` condition on the rotation function's resource policy (Medium), and no alerting on enumeration calls. That last one reflects my lab setup, not a production environment, because I did not configure any monitoring.

The mitigations all come down to narrowing permissions: replace `lambda:*` with specific actions, restrict `iam:PassRole` to named role ARNs with a `PassedToService` condition, remove `AdministratorAccess` from the debug role, use temporary credentials with MFA, and alert on `CreateFunction` and bursts of `List*`/`Get*` calls. After these controls the Critical count drops to zero. R1 still rates High (residual score 10), because the escalation path becomes harder but not impossible, and I think that is the honest result. The likelihood and impact ratings are my own judgement, not measured values, so someone else might score them differently.

Evidence Produced

-PhaseD_risk_matrix.xlsx: risk register (8 risks with likelihood, impact, score, rating, risk statement, mitigation and residual rating), a heat map and a sheet of scales and assumptions.
-PhaseD_threat_model.png: diagram of the privilege escalation path, the mitigations at each step and the existing functions' data paths.

