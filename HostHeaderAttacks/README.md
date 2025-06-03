## Host Header Attacks 

The purpose of the HTTP Host header is to help identify which back-end component the client wants to communicate with. If requests didn't contain Host headers, or if the Host header was malformed in some way, this could lead to issues when routing incoming requests to the intended application. 

It's used to guide to request to the specific severs specific part, Let's say multiple sites are hosted under same IP address / Same server then virtual host helps to distinguish the site which virtual host help to get the data or sometimes it's due to intermediary routining through either CDN or load balancer.

Even if the Host header itself is handled more securely, depending on the configuration of the servers that deal with incoming requests, the Host can potentially be overridden by injecting other headers. Sometimes website owners are unaware that these headers are supported by default and, as a result, they may not be treated with the same level of scrutiny.


## How to test for vulnerabilities using the HTTP Host header
You need to identify whether you are able to modify the Host header and still reach the target application with your request. If so, you can use this header to probe the application and observe what effect this has on the response

## Supply an arbitrary Host header
The first step is to test what happens when you supply an arbitrary, unrecognized domain name via the Host header

The Host header is such a fundamental part of how the websites work, tampering with it often means you will be unable to reach the target application at all. The front-end server or load balancer that received your request may simply not know where to forward it, resulting in an "Invalid Host header" error of some kind. This is especially likely if your target is accessed via a CDN. In this case, you should move on to trying some of the techniques outlined below. 

## Check for flawed validation
Instead of receiving an "Invalid Host header" response, you might find that your request is blocked as a result of some kind of security measure. For example, some websites will validate whether the Host header matches the SNI from the TLS handshake. This doesn't necessarily mean that they're immune to Host header attacks

You should try to understand how the website parses the Host header. This can sometimes reveal loopholes that can be used to bypass the validation. For example, some parsing algorithms will omit the port from the Host header, meaning that only the domain name is validated. If you are also able to supply a non-numeric port, you can leave the domain name untouched to ensure that you reach the target application, while potentially injecting a payload via the port.


```
GET /example HTTP/1.1
Host: vulnerable-website.com:bad-stuff-here
```

 Other sites will try to apply matching logic to allow for arbitrary subdomains. In this case, you may be able to bypass the validation entirely by registering an arbitrary domain name that ends with the same sequence of characters as a whitelisted one: 

```
GET /example HTTP/1.1
Host: notvulnerable-website.com
```

 Alternatively, you could take advantage of a less-secure subdomain that you have already compromised: 


 ```
GET /example HTTP/1.1
Host: hacked-subdomain.vulnerable-website.com
 ```

## Send ambiguous requests
The code that validates the host and the code that does something vulnerable with it often reside in different application components or even on separate servers. By identifying and exploiting discrepancies in how they retrieve the Host header, you may be able to issue an ambiguous request that appears to have a different host depending on which system is looking at it. 

## Inject duplicate Host headers

One possible approach is to try adding duplicate Host headers. Admittedly, this will often just result in your request being blocked. However, as a browser is unlikely to ever send such a request, you may occasionally find that developers have not anticipated this scenario. In this case, you might expose some interesting behavioral quirks. 


```
GET /example HTTP/1.1
Host: vulnerable-website.com
Host: bad-stuff-here
```

## Supply an absolute URL
 Although the request line typically specifies a relative path on the requested domain, many servers are also configured to understand requests for absolute URLs.

The ambiguity caused by supplying both an absolute URL and a Host header can also lead to discrepancies between different systems


```
GET https://vulnerable-website.com/ HTTP/1.1
Host: bad-stuff-here
```

## Add line wrapping

 You can also uncover quirky behavior by indenting HTTP headers with a space character. Some servers will interpret the indented header as a wrapped line and, therefore, treat it as part of the preceding header's value. Other servers will ignore the indented header altogether.

Due to the highly inconsistent handling of this case, there will often be discrepancies between different systems that process your request. For example, consider the following request: 

```
GET /example HTTP/1.1
    Host: bad-stuff-here
Host: vulnerable-website.com
```

## Inject host override headers

Even if you can't override the Host header using an ambiguous request, there are other possibilities for overriding its value while leaving it intact. This includes injecting your payload via one of several other HTTP headers that are designed to serve just this purpose, albeit for more innocent use cases. 

```
GET /example HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-Host: bad-stuff-here
```

Although X-Forwarded-Host is the de facto standard for this behavior, you may come across other headers that serve a similar purpose, including: 

```
X-Host
X-Forwarded-Server
X-HTTP-Host-Override
Forwarded
```



## How to exploit the HTTP Host header

## Web cache poisoning via the Host header

To construct a web cache poisoning attack, you need to elicit a response from the server that reflects an injected payload. The challenge is to do this while preserving a cache key that will still be mapped to other users' requests. If successful, the next step is to get this malicious response cached. It will then be served to any users who attempt to visit the affected page

## Exploiting classic server-side vulnerabilities 
Every HTTP header is a potential vector for exploiting classic server-side vulnerabilities

## Accessing restricted functionality 

For fairly obvious reasons, it is common for websites to restrict access to certain functionality to internal users only. However, some websites' access control features make flawed assumptions that allow you to bypass these restrictions by making simple modifications to the Host header


## Accessing internal websites with virtual host brute-forcing

Companies sometimes make the mistake of hosting publicly accessible websites and private, internal sites on the same server. Servers typically have both a public and a private IP address. As the internal hostname may resolve to the private IP address, this scenario can't always be detected simply by looking at DNS records: 


```
www.example.com: 12.34.56.78
intranet.example.com: 10.0.0.132
```


## Routing-based SSRF 

It is sometimes also possible to use the Host header to launch high-impact, routing-based SSRF attacks. These are sometimes known as "Host header SSRF attacks",

It relies on exploiting the intermediary components that are prevalent in many cloud-based architectures.
Although these components are deployed for different purposes, fundamentally, they receive requests and forward them to the appropriate back-end. If they are insecurely configured to forward requests based on an unvalidated Host header, they can be manipulated into misrouting requests to an arbitrary system of the attacker's choice



## Connection state attacks

For performance reasons, many websites reuse connections for multiple request/response cycles with the same client. Poorly implemented HTTP servers sometimes work on the dangerous assumption that certain properties

For performance reasons, many websites reuse connections for multiple request/response cycles with the same client. Poorly implemented HTTP servers sometimes work on the dangerous assumption that certain properties, such as the Host header, are identical for all HTTP/1.1 requests sent over the same connection. This may be true of requests sent by a browser, but isn't necessarily the case for a sequence of requests sent from Burp Repeater. This can lead to a number of potential issues

## SSRF via a malformed request line

Custom proxies sometimes fail to validate the request line properly, which can allow you to supply unusual, malformed input with unfortunate results.
For example, a reverse proxy might take the path from the request line, prefix it with http://backend-server, and route the request to that upstream URL. This works fine if the path starts with a / character, but what if starts with an @ character instead

```
GET @private-intranet/example HTTP/1.1
```

# Labs 


## Lab: Basic password reset poisoning
1. Go to forget password 
2. Initate the forget password flow
3. Change the host header to collaborator link 
4. You will get the password reset token
5. Reset the token and access carlos account.



## Lab: Web cache poisoning via ambiguous requests

1. Add an additional `Host` header and add exploit server host 
2. You can find that it's updating the script src 
3. Create exploit on the exploit server 

```
alert(document.cookie)
```

over 
```
/resources/js/tracking.js
```


## Lab: Host header authentication bypass 
1. Access the labs `robots.txt` 
2. You can find the `/admin` endpoint
3. Access the endpoint by changing the host to `Host: localhost`
4. Now delete `carlos` by changing the host to `Host: localhost`


## Lab: Routing-based SSRF 
1. Access the lab 
2. Now bruteforce the Host with the following IP address range. (0-255)
    192.168.0.$0$
3. Untick the update host header to match target 
4. You will notice the `302` redirection code.
5. Now visit the site again and update the host to the burteforce Ip address
6. You should be able to access the admin panel.
Now delete the carlos and solve the lab.


## Lab: SSRF via flawed request parsing 

1. Access the page with absolute url 
    ```
    GET https://YOUR-LAB-ID.web-security-academy.net/
    ```
2. In the host field intrude the IP address 192.168.0.0/24
3. Once you get the IP address use that as a host for furhter request
4. Now access /admin with absolute url.
5. Initate the deletion of carlos 
6. Change the request method to GET 
7. Change host 
8. It worked.


## Lab: Host validation bypass via connection state attack

1. Send the Get request to  `/`
2. Try to send to `/admin` with the host `192.168.0.1` 
3. It just redirected
4. Now create a group and add two request one with normal get request another the malicious 
5. You can now iniatiate attack, Additionally keep the connection alive as well. it can be done through the connection header
6. Checking the source code we can initate a deletion of the carlos account and eventually remove the account.


