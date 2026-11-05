Phase B: Setup Homelab / Target Environment

Objective
This phase aimed to deploy a simple Python Lambda function triggered by an S3 bucket event using the AWS Free Tier, and configure it with a least-privilege execution role.

Actions Taken
I created an S3 bucket named `lambda-security-lab-titilayo` in `us-east-1` with all public access blocked. I then created an IAM role named `LambdaExecRole` whose trust policy allows `lambda.amazonaws.com` to assume it. The role has the `AWSLambdaBasicExecutionRole` managed policy for CloudWatch Logging and an inline policy, `S3ReadOnlyLabBucket`, that allows only `s3:GetObject` on the bucket's objects and `s3:ListBucket` on the bucket itself.

I deployed a Python function named `SecureLambdaDemo` using the AWS CLI in AWS CloudShell. The function reads the S3 event and logs the bucket name and object key only, not the object contents. I used the `python3.12` runtime because `python3.9` has reached end of support on Lambda. Finally, I added a resource-based permission allowing `s3.amazonaws.com` to invoke the function, restricted with `--source-arn` and `--source-account`, and configured the bucket to send `s3:ObjectCreated:*` events to the function.

Key Observations
Uploading a test file to the bucket triggered the function, and the CloudWatch log showed the expected message identifying the new object. This confirmed that both permission checks from Phase A work together: the resource-based policy let S3 invoke the function, and the execution role let the function run and write logs using temporary credentials.

Scoping the invoke permission to a specific bucket and account prevents other buckets from triggering the function. Scoping the execution role to a single bucket limits what the function could do if it were misused. The function also logs only metadata, which avoids exposing sensitive data in log files.

I ran into a few practical issues. The default WSL distribution was `docker-desktop`, which has no AWS CLI, and `aws login` did not complete in my Ubuntu environment. I resolved this by running the commands in AWS CloudShell, which uses the console session's credentials. The CLI pager also captured some pasted commands, which I fixed by setting `AWS_PAGER=""` and then verifying that both policies were attached to the role.

Evidence Produced
phaseB_lambda_setup.png is a screenshot of the `SecureLambdaDemo` function configuration showing the S3 trigger and the `LambdaExecRole` execution role.

![Lambda function configuration](phaseB_lambda_setup.png,phaseB_test_log.png)
