## Server Side Request Forgery (SSRF) via Analytics Report


1. Navigate to https://hackerone.com/organizations/ORG/analytics/reports
2. Create new report.
3. Choose some filters.
4. Click on "Apply". [intercept the request in this step] in any "template" field; inject any HTML payload.
5. Now inject an <Iframe> to read internal files as shown in the above POC.

Inject something like 

169.254.169.254/latest/meta-data/hostname
169.254.169.254/latest/meta-data/




