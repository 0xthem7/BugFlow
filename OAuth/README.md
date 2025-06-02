# OAuth 2.0 authentication vulnerabilities

## What is OAuth?
OAuth is a commonly used authorization framework that enables websites and web applications to request limited access to a user's account on another application. Crucially, OAuth allows the user to grant this access without exposing their login credentials to the requesting application. This means users can fine-tune which data they want to share rather than having to hand over full control of their account to a third party. 


## How does OAuth 2.0 work?
OAuth 2.0 was originally developed as a way of sharing access to specific data between applications. It works by defining a series of interactions between three distinct parties, namely a client application, a resource owner, and the OAuth service provider

* **Client application** - The website or web application that wants to access the user's data.
* **Resource owner** - The user whose data the client application wants to access. 
* **OAuth service provider** - The website or application that controls the user's data and access to it. They support OAuth by providing an API for interacting with both an authorization server and a resource server.


OAuth grant types involve the following stages: 
1. The client application requests access to a subset of the user's data, specifying which grant type they want to use and what kind of access they want. 
2. The user is prompted to log in to the OAuth service and explicitly give their consent for the requested access.
3. The client application receives a unique access token that proves they have permission from the user to access the requested data. Exactly how this happens varies significantly depending on the grant type. 
4. The client application uses this access token to make API calls fetching the relevant data from the resource server

## OAuth grant types 
The OAuth grant type determines the exact sequence of steps that are involved in the OAuth process. The grant type also affects how the client application communicates with the OAuth service at each stage, including how the access token itself is sent. For this reason, grant types are often referred to as "OAuth flows". 

There are several different grant types, each with varying levels of complexity and security considerations. We'll focus on the "authorization code" and "implicit" grant types as these are by far the most common. 


## OAuth scopes

For any OAuth grant type, the client application has to specify which data it wants to access and what kind of operations it wants to perform. It does this using the `scope` parameter of the authorization request it sends to the OAuth service.

For basic OAuth, the scopes for which a client application can request access are unique to each OAuth service. As the name of the scope is just an arbitrary text string, the format can vary dramatically between providers. Some even use a full URI as the scope name, similar to a REST API endpoint. For example, when requesting read access to a user's contact list, the scope name might take any of the following forms depending on the OAuth service being used: 

```
scope=contacts
scope=contacts.read
scope=contact-list-r
scope=https://oauth-authorization-server.com/auth/scopes/user/contacts.readonly
```

When OAuth is used for authentication, however, the standardized OpenID Connect scopes are often used instead. For example, the scope openid profile will grant the client application read access to a predefined set of basic information about the user, such as their email address, username, and so on.

## Authorization code grant type

The client application and OAuth service first use redirects to exchange a series of browser-based HTTP requests that initiate the flow. The user is asked whether they consent to the requested access. If they accept, the client application is granted an "authorization code". The client application then exchanges this code with the OAuth service to receive an "access token", which they can use to make API calls to fetch the relevant user data.

All communication that takes place from the code/token exchange onward is sent server-to-server over a secure, preconfigured back-channel and is, therefore, invisible to the end user. This secure channel is established when the client application first registers with the OAuth service. At this time, a client_secret is also generated, which the client application must use to authenticate itself when sending these server-to-server requests

As the most sensitive data (the access token and user data) is not sent via the browser, this grant type is arguably the most secure. Server-side applications should ideally always use this grant type if possible.  
<img src="https://portswigger.net/web-security/images/oauth-authorization-code-flow.jpg">


1. **Authorization request**   
    The client application sends a request to OAuth service `/authorization` endpoint asking for permission to access specific user data.  
    Endpoint mapping maybe different however it can be identified based on the parameter used on the endpoint.

    ```
    GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
    Host: oauth-authorization-server.com
    ```
    This request contains the following noteworthy parameters, usually provided in the query string: 

    
    - ```client_id```  
        Mandatory parameter containing the unique identifier of the client application. This value is generated when the client application registers with the OAuth service.
    - ```redirect_uri```  
        The URI to which the user's browser should be redirected when sending the authorization code to the client application. This is also known as the "callback URI" or "callback endpoint". Many OAuth attacks are based on exploiting flaws in the validation of this parameter. 
    - ```response_type```   
        Determines which kind of response the client application is expecting and, therefore, which flow it wants to initiate. For the authorization code grant type, the value should be **code**.
    - ```scope```
        Used to specify which subset of the user's data the client application wants to access. Note that these may be custom scopes set by the OAuth provider or standardized scopes defined by the OpenID Connect specification
    - ```state``` 
        Stores a unique, unguessable value that is tied to the current session on the client application. The OAuth service should return this exact value in the response, along with the authorization code. This parameter serves as a form of CSRF token for the client application by making sure that the request to its /callback endpoint is from the same person who initiated the OAuth flow

2. **User login and consent**  
    When the authorization server receives the initial request, it will redirect the user to a login page, where they will be prompted to log in to their account with the OAuth provider. For example, this is often their social media account

    It is important to note that once the user has approved a given scope for a client application, this step will be completed automatically as long as the user still has a valid session with the OAuth service. In other words, the first time the user selects "Log in with social media", they will need to manually log in and give their consent, but if they revisit the client application later, they will often be able to log back in with a single click. 

3. **Authorization code grant**
    If the user consents to the requested access, their browser will be redirected to the /callback endpoint that was specified in the redirect_uri parameter of the authorization request. The resulting GET request will contain the authorization code as a query parameter. Depending on the configuration, it may also send the state parameter with the same value as in the authorization request. 

    ```
    GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
    Host: client-app.com
    ```

4. **Access token request**
    Once the client application receives the authorization code, it needs to exchange it for an access token. To do this, it sends a server-to-server POST request to the OAuth service's /token endpoint. All communication from this point on takes place in a secure back-channel and, therefore, cannot usually be observed or controlled by an attacker

    ```
    POST /token HTTP/1.1
    Host: oauth-authorization-server.com
    client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
    ```
    In addition to the ```client_id``` and authorization `code`, you will notice the following new parameters: 

    - `client_secret`  
        The client application must authenticate itself by including the secret key that it was assigned when registering with the OAuth service. 
    - `grant_type`  
        Used to make sure the new endpoint knows which grant type the client application wants to use. In this case, this should be set to authorization_code

5. **Access token grant**  
    The OAuth service will validate the access token request. If everything is as expected, the server responds by granting the client application an access token with the requested scope. 
    ```json
    {
    "access_token": "z0y9x8w7v6u5",
    "token_type": "Bearer",
    "expires_in": 3600,
    "scope": "openid profile",
    …
    }
    ```

6. **API call**
    Now the client application has the access code, it can finally fetch the user's data from the resource server. To do this, it makes an API call to the OAuth service's `/userinfo` endpoint. The access token is submitted in the Authorization: Bearer header to prove that the client application has permission to access this data.
    ```
    GET /userinfo HTTP/1.1
    Host: oauth-resource-server.com
    Authorization: Bearer z0y9x8w7v6u5
    ```

7. **Resource grant**  
    The resource server should verify that the token is valid and that it belongs to the current client application. If so, it will respond by sending the requested resource i.e. the user's data based on the scope of the access token. 

    ```json
    {
    "username":"carlos",
    "email":"carlos@carlos-montoya.net",
    …
    }
    ```
    The client application can finally use this data for its intended purpose. In the case of OAuth authentication, it will typically be used as an ID to grant the user an authenticated session, effectively logging them in. 


## Implicit grant type
It is much simpler as well as much more insecure as there is no back channel for code / token to flow server to server.

1. **Authorization request**  
    The implicit flow starts in much the same way as the authorization code flow. The only major difference is that the response_type parameter must be set to token
    ```
    GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
    Host: oauth-authorization-server.com
    ```

2. **User login and consent**
     The user logs in and decides whether to consent to the requested permissions or not. This process is exactly the same as for the authorization code flow. 
3. **Access token grant**  
    If the user gives their consent to the requested access, this is where things start to differ. The OAuth service will redirect the user's browser to the redirect_uri specified in the authorization request. However, instead of sending a query parameter containing an authorization code, it will send the access token and other token-specific data as a URL fragment. 
    
    ```
    GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
    Host: client-app.com
    ```
4. **API call**  
    Once the client application has successfully extracted the access token from the URL fragment, it can use it to make API calls to the OAuth service's /userinfo endpoint. Unlike in the authorization code flow, this also happens via the browser.
    ```
    GET /userinfo HTTP/1.1
    Host: oauth-resource-server.com
    Authorization: Bearer z0y9x8w7v6u5
    ```
5. **Resource grant**  
 The resource server should verify that the token is valid and that it belongs to the current client application. If so, it will respond by sending the requested resource i.e. the user's data based on the scope associated with the access token.



## OAuth authentication
The basic OAuth flows remain largely the same; the main difference is how the client application uses the data that it receives. From an end-user perspective, the result of OAuth authentication is something that broadly resembles SAML-based single sign-on (SSO).

OAuth authentication is generally impletemented as follow.

1. User chooses to login using social media account, The client application uses the social medias OAuth to get some information about the user, it could be email address that is registered with their account.
2. After receiving the access token, the client application requests this data from resource server. Mainly from the path dedicated `/userinfo` endpoint
3. Once it has received the data, the client application uses it in place of a username to log the user in. The access token that it received from the authorization server is often used instead of a traditional password.



## How do OAuth authentication vulnerabilities arise?

 OAuth authentication vulnerabilities arise partly because the OAuth specification is relatively vague and flexible by design. Although there are a handful of mandatory components required for the basic functionality of each grant type, the vast majority of the implementation is completely optional. This includes many configuration settings that are necessary for keeping users' data secure. In short, there's plenty of opportunity for bad practice to creep in.

One of the other key issues with OAuth is the general lack of built-in security features. The security relies almost entirely on developers using the right combination of configuration options and implementing their own additional security measures on top, such as robust input validation. As you've probably gathered, there's a lot to take in and this is quite easy to get wrong if you're inexperienced with OAuth.

Depending on the grant type, highly sensitive data is also sent via the browser, which presents various opportunities for an attacker to intercept it. 


## Identifying OAuth authentication

If you are able to login into an application using an account of different website / application then this confirm OAuth.

The most reliable way to identify OAuth is to pass the requests through Burp proxy and check the HTTP messages when you are login into your application.

Regardless the grant type the first request is always sent to `/authorization` endpoint containing a number of query parameters that are used specially for OAuth. We should always check for `client_id`, `redirect_uri` and `response_type` parameters. An authorization requests usually looks like 

```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

## Recon

1. Checking the HTTP request flow.
2. If an external service is being used we can check for the hostname.   
*As these services provide a public API, there is often detailed documentation available that should tell you all kinds of useful information* 
We can send request to the following endpoint to gather further information once we know the hostname.
    ```
    /.well-known/oauth-authorization-server
    /.well-known/openid-configuration
    ```

## Exploiting OAuth authentication vulnerabilities

### Vulnerabilities in the OAuth client application
1. The OAuth that is being used by the client application are often hardend and well secured.
2. The issue may be on the implementation of OAuth.
3. As per the grant types there are often multiple flow of HTTP requests which may lead to misconfiguration of the OAuth implementation.


#### Improper implementation of the implicit grant type 


# Labs 

## Lab: Authentication bypass via OAuth implicit flow

1. We logged in using social media and found out that it's using token based OAuth grant type.
2. Now we checked each request and found out that on the endpoint `/authenticate` Along with the access token, username and email is being passed which we than changed it to carlos.
3. Forwarding the request provided access to carlos account.

