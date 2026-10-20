 Phase A: Theory and Conceptual Foundation

Objective

Understand how Amazon S3 stores objects within AWS Regions and Availability Zones, how ACLs differ from bucket policies, and how IAM integrates with S3 access controls.

Activities Completed

- Reviewed official AWS documentation on S3 architecture and Object Ownership.
- Examined the relationship between buckets, objects, IAM roles, IAM policies, bucket policies and ACLs.
- Prepared a relationship diagram.
- Used ChatGPT to summarize the differences between ACLs and bucket policies.
- Identified which access controls provide the most granular permissions.

S3 Architecture

An S3 bucket is created in a specific AWS Region and contains objects. Each object includes its data, metadata and key.

S3 Standard stores objects redundantly across at least three Availability Zones within the selected Region. Single-zone storage classes have different resilience characteristics.

Objects are not automatically replicated to another Region. Cross-Region Replication can be configured to copy objects into a destination bucket in another Region.

S3 Access Control Components

| Component | Purpose |
|---|---|
| IAM role | Provides an assumable identity that uses temporary credentials. |
| IAM permission policy | Defines the actions the role is permitted to perform on specified resources. |
| Bucket policy | Controls access to a bucket and its objects using principals, actions, resources and conditions. |
| Bucket ACL | Provides basic permission grants on a bucket when ACLs are enabled. |
| Object ACL | Provides basic permission grants on an individual object when ACLs are enabled. |

ACLs are disabled by default for new buckets through the Bucket owner enforced Object Ownership setting.

Relationship Diagram

[S3 access control relationships](phaseA_s3_access_diagram.png)

The diagram shows policy relationships. Effective access depends on all applicable controls; an explicit deny overrides an allow.

AI Summary: ACLs vs Bucket Policies

Amazon S3 ACLs provide basic permission grants on buckets or individual objects. They offer limited permissions and do not support explicit deny rules or conditions. ACLs are disabled by default for new buckets.

Bucket policies are JSON resource-based policies that control access to a bucket and its objects. They support specific principals, actions, resources, conditions and explicit deny statements.

IAM policies attached to roles define the S3 actions those roles can perform. AWS evaluates applicable permissions together; an explicit deny overrides an allow.

Bucket policies and IAM policies provide more granular control than ACLs because they can target individual objects and restrict access using detailed actions and conditions. AWS recommends keeping ACLs disabled for most use cases and managing access through policies.

AI tool used: ChatGPT.

Most Granular Access Control

Policy-based controls—bucket policies and IAM policies—provide finer control than ACLs.

For example, a policy can allow a role to read one specific object while requiring requests to use HTTPS. An object ACL can grant access to an individual object but cannot express equivalent conditions.

Key Lessons Learned

- Multi-AZ redundancy and Cross-Region Replication serve different purposes.
- Bucket policies can control access to individual objects, despite being attached to a bucket.
- IAM roles identify requesters; applicable policies determine their access.
- ACLs are a legacy mechanism and are unnecessary for most modern S3 deployments.
- An allow in one policy does not guarantee access when another applicable control denies or restricts the request.

Evidence

- [Relationship diagram](phaseA_s3_access_diagram.png)
- [AI summary text file](phaseA_summary.txt)

References

- [Amazon S3 Overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [S3 Object Ownership and Disabling ACLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/about-object-ownership.html)
- [S3 Data Durability](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html)
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
