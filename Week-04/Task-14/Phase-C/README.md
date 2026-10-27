Phase C - Core Execution: Secure Retrieval via Lambda

Objective
Retrieve a stored secret securely from a Lambda function using the configured least-privilege role.

 Actions Taken
I created a Lambda function named `RetrieveSecretFunction` in us-east-1, using the Python [CONFIRM: 3.12] runtime and the existing role `LambdaSecretAccessRole`. I replaced the default code with a short handler that uses boto3 to call `get_secret_value` on `AppDBCredentials`, parses the JSON secret, and returns only the username. The handler never prints or returns the password. I deployed the function and ran a test event, and the execution succeeded with the response `{"username": "appuser"}`.

```python
import boto3
import json

def lambda_handler(event, context):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId='AppDBCredentials')
    secret = json.loads(response['SecretString'])
    return {'username': secret['username']}



 Key Observations
The function retrieved the secret without any stored access keys, because the role supplied temporary credentials automatically, and the only permission it needed was `GetSecretValue` on one ARN. Returning only the username kept the password out of the test output and the logs, which showed me that how the code handles a secret matters as much as how the secret is stored. I also noticed that a role with no logging permissions would have left no logs to audit, so the logging policy had to be added deliberately.

 Evidence Produced
- `PhaseC_LambdaTest.png`: screenshot of the successful Lambda test showing the response `{"username": "appuser"}`.

 Completion Checklist
- Lambda executes successfully.
- Secret retrieved securely.
- No plaintext credentials logged.

## Next
Phase D
