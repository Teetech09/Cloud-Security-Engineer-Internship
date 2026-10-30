Phase E: Hosted Labs Integration

Objective

The objective of this phase was to reinforce the concepts from Phases A through D using external hosted labs that simulate real-world credential exposure and privilege escalation paths, outside of my own CloudGoat homelab.

 Actions Taken

I completed two labs on PwnedLabs.

Lab 1: Leverage Leaked Credentials for Pwnage. This lab modeled the exact scenario I had built manually in Phases A through D: a `.env` file committed to a repository, containing both database credentials and an AWS access key. I used `aws sts get-access-key-info --profile cloudgoat` to resolve the AWS account ID from the leaked key, then tested the database credentials against the account's IAM sign-in URL. They worked, due to password reuse between the database and IAM accounts, giving me access to the AWS console as that user. From there I opened AWS Secrets Manager and found two secrets: one denied access, and one that returned full RDS connection details. I installed `mariadb-client-core` in WSL2 Ubuntu and connected to the database, where a `flag` table held the lab's completion value.

Lab 2: Path Traversal to AWS Credentials to S3. This lab simulated privilege escalation through a web application vulnerability rather than a committed secret. I scanned the target with `nmap` and found a Flask-based logistics company site. I used `ffuf` with a directory wordlist to discover hidden paths, including an invoice download feature that took a `file` parameter. After registering an account to access it, I tested that parameter for directory traversal and used it to read files outside the intended directory, eventually reaching an AWS credentials file left in a non-default user's home directory on the server. I configured those credentials locally, confirmed them with `aws sts get-caller-identity`, and used them to list and download from the S3 bucket referenced in the site's source code, which held the lab's flag.

 Key Observations

Lab 1 reinforced that the risk from a leaked credential rarely stays contained to the system it was issued for. The exposure started as a database credential pair, but because of password reuse it extended to a full AWS console login, and from there to Secrets Manager and a second database. Each hop widened the blast radius beyond what the original secret appeared to control.

Lab 2 showed a different root cause with a similar ending: instead of a secret being committed, an input validation gap (no sanitization on a file parameter) let me read arbitrary files from the server's filesystem. That eventually surfaced an AWS credentials file that had been left on disk, most likely from a developer running AWS CLI commands directly on.

Evidence Produced
    • PhaseE_lab1_completion.png : Screenshots from the "Leverage Leaked Credentials for Pwnage" lab, showing the resolved AWS account ID and the retrieved value from the `flag` table.
    • PhaseE_lab2_completion.png: Screenshot from the "Path Traversal to AWS Credentials to S3" lab, showing the completed flag retrieval from the S3 bucket.
 
