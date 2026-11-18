Actions Taken

    • Compiled the full flaws.cloud assessment (Levels 1 to 6) into a professional report, Final_Report.pdf. 
    • Documented the complete attack chain from Phases A to D: public S3 listing → authenticated S3 access → secrets in Git history → public EBS snapshot → SSRF to the instance metadata service → IAM, Lambda and API Gateway permission discovery. 
    • Completed the findings template for each level (service, misconfiguration, evidence, C/I/A impact, likelihood, impact, MITRE ATT&CK mapping, remediation, validation method). 
    • Rated each finding by likelihood and impact and ordered them into a prioritized remediation roadmap. 
    • Carried in the Phase E hardening matrix, hardened policy examples, detection plan and sandbox validation results as the defensive section. 
    • Organized every screenshot, command log, redacted output, policy JSON and cleanup proof into one Evidence folder with clear Phase/Level filenames. 
    • Redacted all access keys, secrets and the final challenge solution URL from the report and evidence. 

Key Observations
    • The chain succeeded because of a few repeated root causes: public exposure of storage, poor secrets handling, and over-broad permissions. One weakness in each level handed the attacker the access for the next. 
    • The highest-priority fixes are the ones that break the chain earliest and cheapest: account-level S3 Block Public Access, secret scanning with rotation of exposed keys, snapshot block public access, and IMDSv2 with least-privilege instance roles. 
    • Preventive controls alone are not enough. Detection depends on logging coverage, and object-level S3 access only appears if CloudTrail data events are enabled. 
    • Sandbox validation showed that S3 Block Public Access and EBS snapshot block public access are enforced, challenge credentials were removed locally, and Level 4 resources were deleted. 

Evidence Produced
    • Final_Report.pdf 
    • Single Evidence folder containing: 

