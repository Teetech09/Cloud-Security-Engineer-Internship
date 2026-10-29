Phase D: Incident Response & Credential Rotation

 Objective

This phase aimed to contain the breach from Phase C by revoking the compromised credential, replacing it with a new key stored securely in AWS Secrets Manager, and implementing a preventive control to catch the same mistake before it reaches a repository. I documented the response using the NIST incident response phases: Identification, Containment, Eradication, Recovery and Lessons Learned.

 Action Taken in Phase D

I recorded the whole terminal session with the `script` command so that the commands and their output form the incident log. The task brief used the username `cg_user` and the `cg_leak` profile, but the compromised user in my environment is `manager_cgidndh53tyxvd`. I also carried out the response with my separate lab deployer profile instead of the compromised `cg_leak` key, because in a real incident responders should not act through the credential that has been compromised.

Identification. I listed the access keys of the compromised user with `aws iam list-access-keys` and confirmed that the key found in the repository history in Phase C was still `Active`.

Containment. I deactivated the key with `aws iam update-access-key --status Inactive`, then listed the keys again to confirm the status change. To prove that containment worked, I repeated the `aws sts get-caller-identity` call with the `test_leak` profile that held the leaked key, and it now failed with `InvalidClientTokenId`.

Recovery. I created a new access key for the user and stored it in AWS Secrets Manager as the secret `cg/manager-rotated-key`, piping the key from the `create-access-key` output straight into `create-secret` so that it was never printed to the screen or written to a file. My first attempt failed with a connection error to the Secrets Manager endpoint, which was a temporary network problem. Because the new key's secret value had been discarded from the shell, the key was unusable, so I deleted that orphaned key and listed the keys again before retrying. I then reran the step as a chained command that only clears the variable after `create-secret` succeeds, and it returned the secret's ARN, name and version ID.

Eradication. I deleted the old compromised key with `aws iam delete-access-key` and checked that the user was left with a single active key, the new one.

 Lessons Learned. I added a Gitleaks pre-commit hook that runs `gitleaks protect --staged --verbose` and tested it in a throwaway repository by trying to commit a file containing a made-up AWS-style key. I then ended the recording, removed the account IDs, key IDs and terminal control characters from the log and checked it for leftover secrets before publishing.

 Key Observations

Containment is proven by failure. The clearest evidence of containment was not the status change but the failed authentication. The same key that returned the user's identity in Phase C now returned `InvalidClientTokenId`, which shows that deactivating a key takes effect quickly and stops an attacker holding that key.

Responding with the compromised identity would have been wrong. The task's commands use the leaked profile for the response. Using a separate responder identity avoids depending on, and further using, the credential under investigation. It also means the response is not cut off partway through, since deactivating the key would have removed the access I was relying on.

A failed recovery step created a new risk. The Secrets Manager call failed because of a network problem, and by then the replacement key had already been created. A key whose secret half is not stored anywhere is an orphaned credential: it is active, unused and unrecoverable, so it had to be found and deleted. The lesson is to make rotation steps atomic, so that the new credential is only discarded from memory after it has been stored safely, which is what the chained command does.

Rotation does not undo the exposure. The old key is still present in the demo repository's history, and anyone who copied it before deactivation had a window to use it. Revoking the key ends the risk from that credential, but the response should also include reviewing what the key accessed, which I did not do in this lab and would require CloudTrail in a real incident.

Prevention. The pre-commit hook shifts detection from after a leak to before a commit is created, which is cheaper than rotation. It is a local control that a developer can bypass, so it should be paired with the same scanning in CI and with temporary credentials or managed secrets in place of long-lived access keys.

NIST Incident Response Summary

| Phase | Action |
|---|---|
| Identification | Confirmed the key from the repository history was an active credential of IAM user `manager_cgidndh53tyxvd`. 
| Containment | Set the key to Inactive and verified that it no longer authenticates. |
| Eradication | Deleted the compromised key and the orphaned key from the failed first attempt. |
| Recovery | Issued a new key and stored it in Secrets Manager as `cg/manager-rotated-key`. |
| Lessons Learned | Added a Gitleaks pre-commit hook and noted that rotation steps should store the new credential before discarding it. 

Evidence Produced

* PhaseD_rotation.log: Clean recording of the terminal session covering the key listing, deactivation, the failed authentication test, the Secrets Manager creation and the other response commands, with account IDs and key IDs masked.
* PhaseD_secretmanager.png: Screenshot of the `cg/manager-rotated-key` secret in the AWS Secrets Manager console, with the account ID masked and the secret value not revealed.
