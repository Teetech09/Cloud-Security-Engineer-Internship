
 Phase B: CloudGoat Homelab Setup

 Objective
Deploy CloudGoat's iam_privesc_by_attachment scenario and identify its
IAM users and roles.

 Environment
- AWS CloudShell
- Python 3.13.15
- Python virtual environment: cloudgoat-venv
- Terraform 1.15.9
- CloudGoat installed through pip
- AWS profile: cloudgoat-lab
- Authentication: CloudShell temporary credentials through credential_process

Setup and Deployment
1. Created and activated a Python virtual environment.
2. Installed CloudGoat and Terraform.
3. Configured the cloudgoat-lab AWS profile.
4. Verified the profile using aws sts get-caller-identity.
5. Configured CloudGoat to use the named profile.
6. Created an IP whitelist restricted to CloudShell's detected outbound IP.
7. Deployed the iam_privesc_by_attachment scenario.

Deployment command:

```bash
cloudgoat create iam_privesc_by_attachment

Deployment Result
Terraform reported:
Apply complete! Resources: 16 added, 0 changed, 0 destroyed.

Scenario identifier: cgidj957m3fp3j
Troubleshooting
Issue	Cause or observation	Resolution
Terraform and CloudGoat unavailable	Commands were not installed in the session.	Installed both tools in the Python virtual environment.
Provider installation failed with no space left on device	The home filesystem had insufficient space for the provider installation.	Set TF_DATA_DIR to a temporary directory under /tmp, which had more available space.
Terraform rejected credential_source	Terraform required role_arn for that profile configuration.	Replaced credential_source with a credential_process using AWS CLI credential resolution.
Scenario user missing from identifier-filtered results	The username did not contain the scenario identifier.	Searched for kerrigan and confirmed the user separately.


The temporary Terraform data directory may need to be recreated after a
CloudShell session reset. Scenario files and Terraform state must be
preserved for resource management and cleanup.
IAM Entities Identified
Entity type	Name
User	kerrigan
Role	cg-ec2-meek-role-cgidj957m3fp3j
Role	cg-ec2-mighty-role-cgidj957m3fp3j


These entities were retrieved through AWS CLI enumeration. Their names
alone do not establish their effective permissions.
Evidence
- ../evidence/phaseB_deployed_users.png
- ../evidence/phaseB_deployed_roles.png
- ../evidence/phaseB_provider_disk_space_error.png
- ../iam_entities.csv
Credential Handling
Generated credentials must be excluded from screenshots and repository files.
The CloudGoat start.txt file and Terraform state may contain secrets and
must not be committed.
Outcome
Successfully deployed the scenario and identified one IAM user and two
IAM roles for the entity inventory.



