Phase C: Core Execution, Privilege Escalation Path Discovery

Objective

The objective of this phase was to map the possible privilege escalation paths in the local IAM environment using PMapper, and to visualize the relationships in Neo4j.

Actions Taken

I started in the Moto simulator from Phase B and added two roles to it. `AdminRole` had a policy allowing all actions on all resources, and `LowPrivRole` had a policy that allowed it only to call `sts:AssumeRole` on `AdminRole`. Both roles trusted the account, so the second one could reach the first. Together with `privesc-user` and its `RiskyDevPolicy` from Phase B, this gave the simulated account a low-permission identity and an administrative identity to test against. I then generated the role list with `aws iam list-roles` and saved it as `roles.json`, which showed both new roles.

I installed PMapper from a fork that adds Neo4j support. The packaged install was missing its Neo4j module, so I cloned the source and ran it from a Python virtual environment instead. The commands in the task brief also differed from how PMapper actually works. `graph create` takes the account from the credentials, so it has no `--account` option, and `visualize` produces SVG, PNG or DOT files rather than a `graph.json` file. Neo4j therefore could not simply read a file, so I used the fork's own `neo4j load` command to send the graph straight into the database.

Pointing PMapper at the simulator also needed a workaround. I set the `AWS_ENDPOINT_URL` environment variable so its calls went to Moto, and the first graph build failed because Moto leaves the `PolicyName` field out of the `Policies` entries returned by `get-account-authorization-details`, while real AWS includes it. I confirmed this by inspecting the output, and I patched one line in PMapper so that it falls back to the last part of the policy ARN when the name is missing. After that, `pmapper graph create` completed and mapped three principals.

For Neo4j, I ran the database in Docker. The fork connects with an empty password, so I started the container with authentication disabled, which is acceptable for a local lab database containing only simulated data. I loaded the graph with `pmapper --account 123456789012 neo4j load`, then connected with the Neo4j Browser and ran a query that returns every path from a non-admin principal to an admin principal.

Key Observations

PMapper mapped three principals, two of which it treats as administrative, and it found one edge between them. The privilege escalation query reported that `LowPrivRole` can escalate by calling `sts:AssumeRole` on `AdminRole`, so a role that looks harmless on its own is one step from full control. This is a good example of a chained permission: `LowPrivRole` holds no administrator rights itself, and its escalation comes from a single permission that points at a more powerful role.

PMapper also reported `privesc-user` as an administrative principal, even though it has no policy that grants everything. Its policy allows `iam:PassRole`, `iam:AttachUserPolicy`, `iam:PutRolePolicy` and `iam:CreatePolicyVersion` on all resources, and any of those permissions lets the user give itself administrator access. This matches what Cloudsplaining flagged in Phase A, and it shows that a policy can be effectively administrative without a wildcard action in it.

The graph in Neo4j made the relationships easy to read. The path from `LowPrivRole` to `AdminRole` appeared as two connected nodes with an edge labelled by the `sts:AssumeRole` reason, which is clearer than reading the same information from raw policy documents. The mitigation is to remove or narrow the permission that lets a low-privilege role assume an administrative one, to use conditions on role trust, and to keep IAM write permissions away from everyday identities.

I also want to record the limits of this setup. Moto is a simulator, and it does not behave exactly like AWS, as the missing `PolicyName` field showed. The results are suitable for learning how PMapper models escalation, but a real account would give a fuller picture, because PMapper checks many more services and edge types there.

Evidence Produced

PhaseC_graph.png` is a screenshot of the Neo4j graph showing `LowPrivRole` connected to `AdminRole` through the `sts:AssumeRole` edge. `roles.json` is the role list generated with the AWS CLI. If you also saved a screenshot of the terminal with the PMapper query result, add it here as supporting evidence.
