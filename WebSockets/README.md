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

