Phase C: Core Execution: Anomaly Detection
Objective

The objective of this phase was to detect anomalous IAM and S3 activity in the CloudTrail dataset by using structured Athena queries, checking suspicious source IPs against an external geolocation service, and producing an AI-assisted incident narrative.

Actions Taken

I recreated the Athena table with additional fields (eventSource, userAgent, errorCode and the user ARN) so that S3 events and request origins could be analysed. I ran an IAM anomaly query that grouped CreateUser, AttachUserPolicy and PutUserPolicy events by user, and an S3 query that grouped GetObject requests by source IP. Because the GetObject query returned no rows, I checked which S3 events existed and re-ran the source IP query across all S3 events, which returned the top 20 addresses. I geolocated the top address with ipinfo.io, classified the remaining addresses by range, and wrote an incident narrative in PhaseC_report.txt.

Key Observations

Two identities, Level6 and backup, stood out in IAM activity, creating 17 and 15 users respectively, with frequent policy changes, a recognised persistence and privilege-escalation pattern. A further 19 events had no resolvable user name, which makes attribution harder. On the S3 side, 193.29.252.218 made 23,471 requests and resolved to a hosting provider in Germany with no VPN, proxy or Tor flags. Eight of the top 20 addresses were in non-routable ranges because of the dataset’s anonymisation, so they couldn’t be geolocated. The empty GetObject result showed that object-level logging was not captured, which limits visibility into actual data access.

Evidence Produced

PhaseC_IAM_Anomaly_Query.png	Athena results for the IAM query, with Level6 and backup at the top
PhaseC_S3_GetObject_Query_Empty.png	The GetObject query returning no rows, showing the absence of data events
PhaseC_S3_Event_List.png	S3 event names and counts that do exist in the dataset
PhaseC_IP_Geolocation_Result.png	ipinfo.io result: Sarstedt, Germany, hosting provider, no VPN/proxy/Tor
PhaseC_report.txt	AI-assisted incident narrative with IAM and S3 findings, IP assessments, limitations and recommendations
