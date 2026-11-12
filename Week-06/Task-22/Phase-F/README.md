 Phase F: Documentation & Reporting

IAM and S3 Activity Monitoring and Anomalous Behavior Detection

Objective

The objective of this phase was to produce a professional incident report that summarizes the findings, risks and recommendations from Task 22. The report documents how anomalous IAM and S3 activity can be detected in AWS CloudTrail logs and how a user’s actions can be correlated to attacker behaviour in a post-exploitation scenario. It also covers the detection queries and dashboards used to surface that behaviour, and the use of AI-assisted analysis to identify early indicators of compromise. The findings are presented with a structured incident timeline and a risk matrix.

Actions Taken

I consolidated the work from the earlier phases into one incident report. The scenario was a post-exploitation sequence in which an actor with access to an IAM role used S3 and IAM permission checks to learn the identifiers of the target environment. In the first hosted lab, s3-account-search revealed the AWS account ID that owns a target S3 bucket. In the second, conditional-love used the S3AccessImages role (arn:aws:iam::014498641618:role/S3AccessImages) against the img.cybrlabs.io bucket. It made repeated s3:HeadObject checks against the aws:ResourceOrgID condition key and recovered the Organization ID o-8qt9p6qlo6 one character at a time.

To show how this behaviour would look to a defender, I mapped each attacker step to the CloudTrail event that would record it. Checking the credentials with sts get-caller-identity and enumerating IAM roles correspond to GetCallerIdentity, ListRoles and GetRolePolicy events. The tool’s repeated probing of the bucket corresponds to a burst of HeadObject calls from one identity and one source IP. This mapping is the basis of the correlation between user actions and attacker behaviour in the timeline below.

[Describe your detection work here: the Athena queries you built against the CloudTrail logs (for example a query counting HeadObject calls or denied S3 requests per identity and source IP, and one listing IAM enumeration calls), and the CloudWatch dashboard or widgets you built to visualize them.]

[Describe your AI-assisted analysis here: what log data you gave the AI tool, what indicators of compromise it identified, and how you verified its output against the raw events.]

I ended with prioritized recommendations and a risk matrix, and I noted the CloudTrail review of the lab activity as a follow-up, because the relevant AssumeRole and object-level S3 events have not yet been reviewed.

Incident Timeline
Stage	Attacker behaviour	Expected CloudTrail evidence
1. Access verification	Confirmed the lab credentials worked using the cybr-lab profile	GetCallerIdentity from the lab identity
2. IAM enumeration	Listed roles and identified S3AccessImages	ListRoles and GetRolePolicy events
3. Account ID discovery	Ran s3-account-search against the target bucket	S3 access attempts against the bucket (object-level events appear only if data events are enabled)
4. Organization ID discovery	Ran conditional-love with s3:HeadObject and aws:ResourceOrgID	AssumeRole followed by a burst of HeadObject calls from one identity and source IP, with a mix of allowed and denied results
5. Result	Recovered account ID and Organization ID o-8qt9p6qlo6	No single event shows the disclosure; it is detected through the pattern in stage 4

[Add the actual event timestamps, identities and source IPs once you have reviewed them in CloudTrail Event History or Athena.]

Key Observations

The main observation is that AWS identifiers which are not secrets can still be discovered through normal permission evaluation. S3 bucket access was enough to reveal the owning account ID, and the aws:ResourceOrgID condition key worked as an information-disclosure oracle. Each guess was allowed or denied depending on whether it matched the start of the real Organization ID, which turned a search of every possible ID into a short, predictable process.

For detection, no single event in the sequence is clearly malicious. A HeadObject call or an IAM role listing is normal on its own. The anomaly is the pattern: many HeadObject calls in a short time from one identity, a mix of allowed and denied responses, and IAM enumeration just before it. This is why the detection queries and dashboards focus on volume, frequency and identity grouping, and not on individual events. It also means detection depends on logging. S3 object-level activity appears in CloudTrail only when data events are enabled, so a trail without them would miss the probing entirely.

On risk, the exposure is a reconnaissance weakness and not a direct compromise. Its impact depends on how trust policies, cross-account access and organization-scoped conditions are configured, and it grows wherever those controls are loose.

Risk Matrix
Finding	Likelihood	Impact	Rating
Account ID discoverable through S3 bucket access	High	Low	Low–Medium
Organization ID discoverable through aws:ResourceOrgID condition evaluation	Medium	Medium	Medium
S3 object-level activity not logged (data events disabled)	Medium	High	Medium–High
Overly broad S3 permissions on roles	Medium	Medium	Medium

The ratings are my own assessment, based on how easily each issue could be exploited and what an attacker could do with the result.

Recommendations
Apply least-privilege permissions to IAM roles and restrict S3 access to the specific buckets and actions needed.
Review resource policies and the use of global condition keys such as aws:ResourceOrgID.
Restrict unnecessary cross-account access and review role trust policies.
Enable CloudTrail data events for sensitive S3 buckets so object-level activity is visible.
Alert on bursts of HeadObject or denied S3 requests from a single identity or source IP, and on IAM enumeration followed by S3 probing.
Review AssumeRole events for unusual role use.


Evidence Produced
PhaseF_IAM_S3_Anomaly_Report_YNT.pdf. —Final Report
      

      
