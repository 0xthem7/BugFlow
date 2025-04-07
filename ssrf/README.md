# What is SSRF 
Server-side request forgery is a web security vulnerability that allows an attacker to cause the server-side application to make requests to an unintended location.

* It's something like requesting the request from the server to the server

# Why does SSRF works 
* Lack check implementation as its on the different componenet which sits in front of application server, When a connection is made back to the server the check is bypassed.
* It assumes only a fully trusted user would come directly from the server


Lab: 1. Basic SSRF against the local server
  - Send a request something like stockapi=http://localhost/admin/
  *Pro tip: Just send request to localhost*

* You try to access internal data of other backend system as well 
  - 192.168.1.1/24

Lab: 2. Basic SSRF against another back-end system 
  1. Try to find the open address and ports over the srver
  2. Use burp intruder to automate the process
  4. Try to bruteforce the port as well
  http://192.168.0.124:8080/admin/delete?username=carlos

  *Pro tip: Bruteforce every port and host out there*

Lab: 3. SSRF with blacklist-based input filter
  1. Try every methods mentioned over [Bypass Methods](#some-way-to-bypass)
  2. Use the different combination of encoding and upper lower case everything
  http%3a//127.0.1/Admin/delete?username=carlos

  *Pro tip: Use different methods of encoding it should only have to work once ;)* 


Lab: 4. SSRF with filter bypass via open redirection vulnerability
  1. Send the request to product/nextProduct?currentProductId=1&path=http://localhost.himansu.com.np/admin/delete?username=carlos
  2. Firstly find a open redirect 
  3. Combine it with normal request (Lets say checking stockapi)


Lab: 5. Blind SSRF with out-of-band detection
  1. As the application is using referer header to do some analytics with analytic software
  2. Append payload on the refferer header 
  3. Colaborator should recieve http requests



### SSRF with whitelist-based input filters
Some applications only allow inputs that match, a whitelist of permitted values. The filter may look for a match at the beginning of the input, or contained within in it. You may be able to bypass this filter by exploiting inconsistencies in URL parsing

*  You can embed credentials in a URL before the hostname, using the @ character. For example: 
```https://expected-host:fakepassword@evil-host```
*  You can use the # character to indicate a URL fragment. For example: 
```https://evil-host#expected-host```
* You can leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control. For example: 
```https://expected-host.evil-host```
* You can URL-encode characters to confuse the URL-parsing code. This is particularly useful if the code that implements the filter handles URL-encoded characters differently than the code that performs the back-end HTTP request. You can also try double-encoding characters; some servers recursively URL-decode the input they receive, which can lead to further discrepancies. 
*  You can use combinations of these techniques together. 

### Bypassing SSRF filters via open redirection

You can leverage the open redirection vulnerability to bypass the URL filter, and exploit the SSRF vulnerability as follows
```stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin```
 This SSRF exploit works because the application first validates that the supplied stockAPI URL is on an allowed domain, which it is. The application then requests the supplied URL, which triggers the open redirection. It follows the redirection, and makes a request to the internal URL of the attacker's choosing.




### Some ways to bypass the restrictions (#some-way-to-bypass)
1. Use an alternative IP representation of 127.0.0.1, such as 2130706433, 017700000001, or 127.1 
  ```127 * 256^3 + 0 * 256^2 + 0 * 256 + 1 = 2130706433```
  ```http://2130706433 == http://127.0.0.1```
  
  * Integer Format - 2130706433

  * Octal Format – 017700000001
    Each octet is in octal:

    127 in octal: 0177

    0 in octal: 000

    0 in octal: 000

    1 in octal: 001

  comombined: 017700000001

  * Abbreviated Format – 127.1
    You can abbreviate IPs:

    127.1 → 127.0.0.1

    127.0.1 → 127.0.0.1 (in some contexts)
  
2. Register your own domain name that resolves to 127.0.0.1 - **localhost.himansu.com.np**
3. Obfuscate blocked strings using URL encoding or case variation. 
4. Provide a URL that you control, which redirects to the target URL
  * Try using different redirect codes, as well as different protocols for the target URL. For example, switching from an http: to https: URL during the redirect has been shown to bypass some anti-SSRF filters.


### Blind SSRF vulnerabilities 
Blind SSRF vulnerabilities occur if you can cause an application to issue a back-end HTTP request to a supplied URL, but the response from the back-end request is not returned in the application's front-end response

How to find and exploit blind SSRF vulnerabilities
* The most reliable way to detect blind SSRF vulnerabilities is using out-of-band (OAST) techniques
  This involves attempting to trigger an HTTP request to an external system that you control, and monitoring for network interactions with that system. 

### Finding hidden attack surface for SSRF vulnerabilities
1. Partial URLs in requests 
2. URLs within data formats 
3. SSRF via the Referer header
  * Some applications use server-side analytics software to tracks visitors. This software often logs the Referer header in requests, so it can track incoming links Often the analytics software visits any third-party URLs that appear in the Referer header. This is typically done to analyze the contents of referring sites, including the anchor text that is used in the incoming links. As a result, the Referer header is often a useful attack surface for SSRF vulnerabilities. 



## Writeups analysis
[SSRF reports Analysis](./reports/README.md)
