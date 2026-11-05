Phase E: Hosted Labs Extension

Objective

The objective of this phase was to reinforce the concepts from this task using an external hosted lab simulating a related cloud attack vector: credential exposure through a misconfigured resource, this time a public EBS snapshot rather than the metadata service used in Phases A through D.

Actions Taken

I completed the PwnedLabs lab **Loot Public EBS Snapshots**, starting with the low-privileged IAM credentials PwnedLabs provisions for the `intern` user. Running `aws sts get-caller-identity` and `aws iam get-policy-version` on the attached policy, `PublicSnapper`, showed the intern's access was scoped to a single snapshot, `snap-0c0679098c7a4e636`, with permission to describe it and list EBS blocks directly via the EBS Direct API.

My first approach was to pull the snapshot's contents using that direct API, writing a Python script with `boto3` to call `list_snapshot_blocks` and `get_snapshot_block` for all 16,384 blocks and reassemble them into a raw disk image. This worked and is a valid technique, but reading PwnedLabs' own walkthrough afterward showed the lab's intended path was simpler: since the snapshot itself is public, it can be restored to volume and mounted in any AWS account, including my own, which I then did instead.

Using my own AWS account, I confirmed the snapshot's `createVolumePermission` showed `Group: all`, confirming it was genuinely public, and that it was unencrypted, both required for this exposure to be exploitable. I created a volume from the snapshot, launched a `t3.micro` EC2 instance in the same availability zone, attached the volume, and SSH'd in. After mounting the volume at `/dev/nvme1n1p1`, I found `/home/intern/practice_files/s3_download_file.php`, which contained hardcoded AWS credentials for a user with access to a private S3 bucket, `ecorp-client-data`. Using those stolen credentials, I listed and read the bucket's contents, including the lab's flag file, which I submitted to PwnedLabs to mark the lab complete.

Key Observations

This lab demonstrated that an EBS snapshot is a full point-in-time copy of a disk, so making one public exposes everything that was ever written to it, not a single file or setting. The actual secret in this lab was sitting in an application source file that had simply been included in the backed-up disk image, which is a realistic way for credentials to leak, since snapshot creation is routine automation that most people don't think of as a credential exposure surface.

Trying the EBS Direct API route first, before reading the walkthrough, was a useful exercise in its own right. It is a legitimate and sometimes necessary technique, for example when an attacker's access doesn't extend to creating volumes or instances in their own account, but it is also far slower and more complex than simply restoring the snapshot when that option is available. Knowing both paths is useful: the direct block API is the fallback when volume creation isn't possible, and volume restoration is the direct route when it is.

The permission structure of the `PublicSnapper` policy was also worth noting: it granted `ec2:DescribeSnapshots` without restricting it to a specific resource, while `ec2:DescribeSnapshotAttribute` was scoped to one snapshot ID. This meant the intern could discover the existence of every snapshot in the account but only inspect the sharing permissions of the one the lab intended, which is a reminder that broad list/describe permissions can still leak more than intended even when a related action is tightly scoped.

Evidence Produced

The evidence for this phase is PhaseE_lab.png, a screenshot of the PwnedLabs completion screen confirming the lab was finished after the flag was submitted.
