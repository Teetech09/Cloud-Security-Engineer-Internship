Phase D: Encryption and Misconfiguration Simulation

Objective

Verify S3 server-side encryption and demonstrate how access-control misconfigurations can expose an encrypted object.

Resources Used

| Resource | Purpose |
|---|---|
| student-s3-lab-teetech-20261020 | Original private lab bucket; used for encryption verification |
| encrypted_data.txt | Sample object used to verify SSE-S3 |
| student-s3-public-sim-teetech-20261020 | Separate bucket used for temporary public-access testing |
| public-demo.txt | Harmless dummy object used in the simulation |

1. Encryption Verification

I confirmed that the original lab bucket had default encryption configured as SSE-S3.

I created and uploaded a sample file:

~~~powershell
"Simulated lab data only. Testing S3 server-side encryption." | Set-Content encrypted_data.txt

aws s3 cp encrypted_data.txt s3://student-s3-lab-teetech-20261020/encrypted_data.txt --profile s3tester
~~~

I opened the object's properties in the S3 console and confirmed that it used server-side encryption with Amazon S3 managed keys (SSE-S3).

2. Simulation Setup

I created a separate general purpose bucket with:

- SSE-S3 encryption enabled.
- ACLs disabled.
- Versioning disabled.
- Block All Public Access initially enabled.

Only harmless dummy data was uploaded:

~~~powershell
"PUBLIC ACCESS SIMULATION - dummy data only. No sensitive information." | Set-Content public-demo.txt
~~~

I uploaded the file through the AWS Console because the s3tester profile was scoped to the original lab bucket.

 3. Private Baseline

I opened the object's direct S3 Object URL in a private browser window.

**Result:** AccessDenied.

No presigned URL was used.

4. Temporary Public-Read Configuration

On the simulation bucket only, I retained the two ACL-related public access blocks and temporarily disabled the two policy-related public access blocks.

I applied this bucket policy:

~~~json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "TemporaryPublicReadDummyObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::student-s3-public-sim-teetech-20261020/public-demo.txt"
    }
  ]
}
~~~

The policy allowed anonymous reading of only public-demo.txt. It did not grant bucket listing, uploading or deletion.

Result:** The dummy object became accessible through its Object URL without AWS authentication.

5. Restoration and Verification

Immediately after testing, I:

1. Deleted the temporary public-read bucket policy.
2. Re-enabled all four Block Public Access settings.
3. Retested the Object URL in a fresh private browser window.

Result:** AccessDenied.

The original lab bucket's access controls were unchanged.

Test Results

| Stage | Anonymous Object Read |
|---|---|
| Before misconfiguration | Denied |
| During temporary public-read configuration | Allowed |
| After restoring controls | Denied |

Risk Matrix

| Asset | Threat / Misconfiguration | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| S3 bucket data | Public-read policy combined with relaxed Block Public Access settings permits anonymous downloads. | High if sensitive data is exposed; only dummy data was used in this test. | Medium; qualitative assessment for the simulated scenario. | Enforce Block Public Access, remove public grants, apply least privilege and monitor policy changes. Use SSE-S3 or SSE-KMS for encryption at rest. |

Key Observations

SSE-S3 protects data at rest but does not prevent anonymous downloads when access permissions allow them.

The simulation demonstrated a confidentiality risk arising from public-read permissions combined with relaxed protective settings.

The policy granted only GetObject, so this test did not demonstrate public uploading, deletion or bucket listing.

Restoring Block Public Access and deleting the temporary policy blocked anonymous reading again.

SSE-KMS can provide additional key-level controls, but it does not replace appropriate S3 access controls.

 Evidence

- [Bucket encryption settings](PhaseD_bucket_encryption.png)
- [Object encryption verification](PhaseD_object_encryption.png)
- [Private baseline](PhaseD_before_public_access.png)
- [Temporary public access](PhaseD_public_read_success.png)
- [Private access restored](PhaseD_private_access_restored.png)
- [Risk matrix](phaseD_riskmatrix.pdf)

 Completion Checklist

- [x] SSE-S3 encryption verified.
- [x] Misconfiguration simulated using harmless data.
- [x] Temporary public-read policy removed.
- [x] Block All Public Access restored.
- [x] Anonymous access denial verified.
- [x] Risk matrix documented.
