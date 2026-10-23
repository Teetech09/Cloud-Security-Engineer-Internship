Phase B: Setup Homelab Environment

Objective
Deploy a misconfigured S3 environment using CloudGoat and verify it with the AWS CLI.

Environment
- Host: Windows 10 laptop running Ubuntu on WSL2
- AWS CLI v2.38.4
- Terraform v1.17.0
- CloudGoat 2.5.0 (installed with pipx)
- AWS sandbox account, dedicated IAM lab user, CLI profile `cloudgoat`

Scenario used
`cloud_breach_s3`

> The brief names `s3_buckets`, which is not available in CloudGoat 2.5.0. `cloud_breach_s3` is the S3-focused scenario that fits the objective.

Steps performed
1. Installed WSL2 Ubuntu, AWS CLI v2 and Terraform.
2. Created a dedicated IAM user for the lab and configured the AWS CLI profile `cloudgoat`.
3. Verified credentials:
   ```bash
   aws sts get-caller-identity --profile cloudgoat
   ```
4. Installed and configured CloudGoat:
   ```bash
   pipx install cloudgoat
   cloudgoat config aws
   cloudgoat config whitelist --auto
   cloudgoat list
   ```
5. Deployed the scenario:
   ```bash
   cloudgoat create cloud_breach_s3
   ```
6. Verified the environment:
   ```bash
   aws s3 ls --profile cloudgoat
   ```

 Result
Bucket created by CloudGoat: `cg-cardholder-data-bucket-cgidy6z70iw9n5`

![S3 bucket listing](phaseB_s3_lists.png), PhaseB_Cloudgoat_Installed.

Completion checklist
- [x] CloudGoat deployed successfully
- [x] Buckets listed via AWS CLI
- [x] Environment verified operational

Challenges and fixes
- **WSL network issue:** downloads failed over IPv6. Forcing IPv4 (`curl -4`) and setting IPv4 preference fixed it.
- **CloudGoat install changed:** the current version installs with `pipx` and is run as `cloudgoat`, not `./cloudgoat.py`.
- **Scenario name:** `s3_buckets` was not available, so I used `cloud_breach_s3`.

 Cleanup
- [x] Ran `cloudgoat destroy cloud_breach_s3`
- [x] Confirmed the bucket is gone with `aws s3 ls`
- [x] Deleted the lab IAM user's access key

