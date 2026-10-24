Phase B: Homelab / Target Setup

Objective
Deploy a controlled vulnerable AWS environment using CloudGoat for IAM security analysis.

Tools and Environment
- Windows PowerShell
- Ubuntu on WSL2
- AWS CLI
- CloudGoat
- AWS CLI profile: `cloudgoat`
- Scenario: `iam_privesc_by_rollback`

Actions Taken

1. Opened Ubuntu
I checked the available WSL distributions and found that Docker Desktop was the default. I explicitly opened Ubuntu to access my existing lab tools.

```powershell
wsl -l -v
wsl -d Ubuntu
```

2. Located the Existing CloudGoat Installation
I searched my Ubuntu home directory for CloudGoat.

```bash
find ~ -maxdepth 4 -type d \( -iname '*cloudgoat*' -o -name 'cloudgoat-venv' \) 2>/dev/null
```

The installation was located at:

```text
/home/teetech/cloudgoat
```

I reused this installation instead of cloning the repository again.

3. Verified the AWS Profile and Identity
I listed the configured AWS CLI profiles.

```bash
aws configure list-profiles
```

The available profile was:

```text
cloudgoat
```

I confirmed the identity associated with that profile.

```bash
aws sts get-caller-identity --profile cloudgoat
```

The command returned a user ID, account ID and ARN, confirming that authentication worked.

 4. Deployed the CloudGoat Scenario
I navigated to the CloudGoat directory and deployed the rollback scenario.

```bash
cd ~/cloudgoat
cloudgoat create iam_privesc_by_rollback --profile cloudgoat
```

The deployment completed without errors.

Scenario Architecture
The scenario is designed to create:
- One IAM user named Raynor, with a deployment-specific username.
- An IAM managed policy containing five policy versions.

The intended vulnerability involves permission to change the default policy version. An older policy version grants broader permissions, creating a potential privilege-escalation path.

 Key Observations
- Docker Desktop and Ubuntu are separate WSL distributions. Selecting Ubuntu explicitly provided access to my existing CloudGoat installation.
- The `cloudgoat` AWS CLI profile authenticated successfully.
- The existing CloudGoat installation was reused, and the scenario deployed without errors.
- Policy version management is security-sensitive because restoring an older policy can reintroduce excessive permissions.
- Deployment success confirms environment creation; it does not prove that privilege escalation has been performed.

 IAM User Verification
After deployment, the following command can be used to list the scenario user:

```bash
aws iam list-users --profile cloudgoat \
  --query "Users[?contains(UserName, 'raynor')].{UserName:UserName,Arn:Arn}" \
  --output table
```

The resulting username and ARN should be recorded as evidence for later analysis.

 Evidence
Required screenshot:

- `phaseB_setup.png` — Terminal output showing the created Raynor IAM user.

Upload the screenshot after running the verification command. Generated credentials must be excluded from the image.

Completion Status
- [x] Existing CloudGoat installation located.
- [x] AWS deployment identity verified.
- [x] Scenario deployed without errors.
- [x] Created IAM user confirmed through the AWS CLI.
- [x] Evidence screenshot saved and uploaded.

Credential Handling
Generated access keys and secret access keys are kept out of this repository. AWS credential files, CloudGoat output containing secrets and Terraform state files must not be uploaded.

Reference
[CloudGoat: IAM Privilege Escalation by Rollback](https://github.com/RhinoSecurityLabs/cloudgoat/tree/master/cloudgoat/scenarios/aws/iam_privesc_by_rollback)
