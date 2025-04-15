### Server Side Request Forgery mitigation bypass

https://hackerone.com/reports/632101

Not much interesting 

*The validate function performs DNS lookup to check whether the IP address of a domain belongs to the local network*

Steps to reproduce

1. Create a webhook for a repository on GitLab.com. Use the URL http://990.hacker1.xyz. It may return error but let's ignore it now.
2. Wait about 10 seconds and test webhook by clicking on "Test" and "Push events".
3. After the hook has executed, you should see content of http://169.254.169.254 returned.

