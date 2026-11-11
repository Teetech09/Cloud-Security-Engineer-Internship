Phase B: Setup Homelab Environment

Objective

The objective of this phase was to prepare a secure local and cloud environment for querying and visualising AWS CloudTrail logs, using a public CloudGoat-style attack dataset loaded into Amazon S3 and analysed with Amazon Athena.

Actions Taken

I downloaded the public flaws.cloud CloudTrail dataset (about 240 MB), which contains anonymised attack activity captured over several years, and extracted its 20 compressed log files in a WSL Ubuntu environment. The logs were stored in the standard CloudTrail format, where every file wraps its events in a single Records array. Because the Athena JSON SerDe expects one JSON object per line, I flattened the files with jq into a single newline-delimited file of about [X] events, then compressed it with gzip. I created a private S3 bucket named cg-log-analysis-titilayo in us-east-1 and uploaded the compressed log file to it (about 251 MB), confirming the upload with aws s3 ls. I also created a separate results bucket so that Athena query output would not mix with the source logs.

In the Athena console, I configured the query result location, created a database named cg_logs, and created an external table called cloudtrail_logs using the OpenX JSON SerDe, with columns for eventVersion, userIdentity, eventTime, eventName, awsRegion, sourceIPAddress and requestParameters. The table was created without errors. I then ran an aggregation query that groups events by eventName and counts them, to verify that the data was readable and correctly parsed.

Key Observations

The first finding was a data-format issue. The raw CloudTrail files could not be queried directly with the specified SerDe because of the Records wrapper, so flattening the data was a necessary preprocessing step. Without it, the table would have been created successfully but returned empty or NULL results, which shows that a successful CREATE TABLE does not by itself prove the data is queryable.

The verification query returned valid results across [X] distinct event types, with [top event names, e.g. ListBuckets, GetObject] appearing most frequently. The mix of events included a range of IAM and S3 activity, which gives a realistic basis for the threat-hunting work in later phases. I also kept both buckets private and stored query results separately from the source data, which is a basic security practice for log analysis environments.

Evidence Produced

PhaseB_query.png shows the Athena query editor with the eventName aggregation query, its Completed status, and the results table, with the cg_logs database and cloudtrail_logs table visible in the left panel. A second screenshot of the aws s3 ls output confirms the dataset upload to cg-log-analysis-titilayo.
