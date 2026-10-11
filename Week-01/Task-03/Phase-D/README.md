Phase D — Temporary Credentials and MFA

Objective
Generate MFA-authenticated temporary credentials, configure a temporary AWS CLI profile, and verify its identity and expiration.

Actions Taken
- Retained the root account MFA configured in an earlier task.
- Assigned an authenticator-app MFA device to `ops-user`.
- Requested a one-hour session through AWS Security Token Service (STS).
- Configured the returned credentials in the `ops-temp` profile.
- Verified the temporary profile’s AWS identity.

MFA Setup
In the AWS console, I navigated to:

IAM → Users → ops-user → Security credentials → Assign MFA device**

I selected **Authenticator app**, scanned the QR code, and entered two consecutive codes to complete registration. I used the MFA device ARN assigned to `ops-user` for the session request.

Session Generation
In PowerShell, I entered the device ARN and a fresh MFA code:

```powershell
$mfaArn = Read-Host "Enter ops-user MFA device ARN"
$mfaCode = Read-Host "Enter current MFA code"
```

I captured the session response without displaying its credentials:

```powershell
$sessionJson = aws sts get-session-token --serial-number $mfaArn --token-code $mfaCode --duration-seconds 3600 --profile ops-user --region us-east-1 --output json --no-cli-pager

$mfaSession = $sessionJson | ConvertFrom-Json
```

Temporary Profile Configuration
I saved all three temporary credential components:

```powershell
aws configure set aws_access_key_id $mfaSession.Credentials.AccessKeyId --profile ops-temp
aws configure set aws_secret_access_key $mfaSession.Credentials.SecretAccessKey --profile ops-temp
aws configure set aws_session_token $mfaSession.Credentials.SessionToken --profile ops-temp
aws configure set region us-east-1 --profile ops-temp
aws configure set output json --profile ops-temp
```

No credential values are included in this documentation.

Validation
I displayed the expiration and checked the temporary profile’s identity:

```powershell
Write-Output "Token expiration: $($mfaSession.Credentials.Expiration)"
aws sts get-caller-identity --profile ops-temp --region us-east-1 --no-cli-pager
```

The session response showed:

```text
Token expiration: 2026-
