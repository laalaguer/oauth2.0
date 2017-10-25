Oauth 2.0

# Problem:
* Password compromise.
* No specific restrict on content available.
* No specific restrict on duration of visit.
* No revoke ability except change password.

# Oath 2.0:
Used with HTTP/HTTPS only.

# Roles:
1. Client - the agent, browser, app, desktop application, any applicatio that acts on behalf of a resource owner.
2. Resource owner - the content owner. Human user, etc.
3. Authentication Server - the place where stores user/user credentials. Issue access token to client.
4. Credentials - The secret shared between server and user.
5. Resource Server - the place where stores content. Eg. music, files, health data about a user etc.

# Flow:
1. Client asks Resource Owner for (Authorization)
2. Resource Owner grants (Authorization)
3. Client asks Authorization Server with (Authorization) for an (Access Token)
4. (Access Token) is granted.
5. Client asks Resource Server with (Access Token) for a resource.
6. Resource is transmitted.

Step 3-6 is quite straight forward, step 1-2 can vary into 4 forms.

# Four Types of (Authorization):
## 1. Access Code (Twitter app example)

Most secure flow.

Client redirects user to authorization server to get access code.
User redirects back to client and input the access code into client.
Client use access code to ask Authorization server for access token.

Merit:
* a) Client doesn't know the user credential/password.
* b) User doesn't know the access token.
* c) Server can check client type.

Demerit:
* a) Flow long, access code - then exchange for access token.
* b) User expierence not so smooth. - why user need to pass access token around? They want an expierence similar to direct login.

## 2. Implicit (Browser app example)
Simplified flow, used in browser.
Client redirect user to authorization server.
User authorize and redirected back to client with access token already.

Merit:
* a) Simplified flow.
* b) Mostly on web browser.

Demerit:
* a) Server only check client by redirect_URI
* b) Access token is available to user and also anyone who have access to the client.

## 3. Password Credentials (Direct authorization)
Client get username/password directly from user, and exchange it for an access token.

Merit:
* a) One time password fly in the internet.

Demerit:
* a) Password leaking to the client.

# 4. Client Credentials
Client == Resource owner. This is very rare to see.

Credentials are issued to the client via another channel previously.

# Access Token
A string. Issued by authorization server, hold by client, verified by resource server.

Can be a random string, or self-contained info of scope, signature. The details of a secure token format is discussed in [RFC6750]

# Refresh Token
A string. Issued along with access token, but optional. Used to request for a new access token if current access token expires.

# HTTP Redirect
Used in example: 302.

# Client Registration (APP registration)
Client registration 3 infomation:
1. Client type
2. Client redirect URI
3. Developer Miscellaneous

## Client Type
1. Secure: Client credentials can store securely. eg. Server with limited access to admins. Credential can be stored securely on the server code files.
2. Public: Browser app, Desktop app, app on smart phone, etc. Where they cannot guarantee the confidentiality of secrets.

## Typical Clients
1. Web Application: A server. User access the server with HTML interface. Access token and client credentials are stored in server DB and not touchable by end user/any other parties. -- Secure client.
2. User Agent application: A browser. Resource owner can see the client credentials. -- Public client.
3. Native application: An app on smart phone, an app on Desktop. client credentials can be extracted. Some protection of access token.

## Client Identifier, Client Authentication
The server decides whether the client is good or not. Basic Client authentication has two parts: client identifier and client secret.`client_id` and `client_secret`.

The authentication of client is done by providing both `client_id` and `client_secret`. The supported format is:
1. [RFC2617] defined basic authorization (MUST):

`Authorization: Basic czZCaGRSa3F0Mzo3RmpmcDBaQnIxS3REUmJuZlZkbUl3`

In here the username:password is "application/x-www-form-urlencoded" and send to the server side in the `Authorization` header.

2. Request Body (MAY):

`POST` method `client_id=xxx` and `client_secret=yyy`. Note: Do not include the above information inside the URI for example: `GET` `?client_id=xxx&client_secret=yyy`. It is easily leaking to a third party website via `http-forworded-for` section.

# Protocol Endpoints
## Authorization Endpoint
Authorization endpoint verify the resource owner, and issue a response of verification. Depends on the `flow` that client choose, `access code` or `access token` is issued if verification is successfully done.

`response_type` must be included in the client request, to indicate which type of grant it wants. 

## Redirect Endpoint
After the verify step, resource owner is redirected to the redirect endpoint URI which is a registered URI during the registration phase, and only to the pre-registered URI. 

Usually the clients are public clients and confidential clients using a "implicit" grant type.(90% of cases) and those redirect URI MUST be pre-registered. Client can register multiple redirect URIs.

`redirect_uri` can be included in the client request, authorization server must compare the URI with the registered URI.

HTTPS is recommended in this step as `access token` or `access code` is transmitted. 

If the redirect URI is not passing the critieria above, no re-direct performed on the authorization server. Flow ends here. And a message of error should be informed to resource owner.

Redirect endpoint would get the `access_code` or `access_token` during the end of successful redirect, so please execute JS carefully and no third-party JS execution before the desired process of access_code is executed.

**Note: Redirect URI must be absolute, which means no # is allowed **

## Token Endpoint
Server to issue the `access token`. Only used in `access code` flow and refresh phase. Must support the POST method. When requiring the access token, client must use POST method to avoid leaking of sensitive information (refresh token or access code)

Token endpoint must verify the client credential:
1. `Access code`/`refresh_token` vs client pair. The issued token is to the specific client.
2. When a client is compromised, it is easier to invoke the client credential rather than invoke a bunch of refresh token.

## Access Token Scope
The service should specify the scope (resource sets) that the client can visit. It should be expressed in `scope` parameter in request. Also if the server agrees or partially deines the scope, then shall return a `scope` parameter in response.

# Authorization
Four types of authorization is supported in Oauth 2.0. "authorization code", "Implicit", "resource owner password credentials" and "client credentials". 

Most used 2 types are: "Authorization code" which is in app(client is the app itself), or "Implicit" which is used in web browser (the client has a server).

## Authorization Code Mode
```
     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)

   Note: The lines illustrating steps (A), (B), and (C) are broken into
   two parts as they pass through the user-agent.
```

This mode is optimized for secure clients. Eg. A server client which can keep secrets. The steps of authentication is pictured as below:

1. Client redirects resource owner to the authentication endpoint. To request an authorization. The parameters must include, for example:
```
GET /authorize?
response_type=code
&client_id=s6BhdRkqt3
&state=xyz
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

response_type MUST be code
client_id MUST be registered
redirect_uri OPTIONAL but if registered can be ommited.
scope OPTIONAL
state OPTIONAL like a form hash, to maintain the conversation to avoid CSS attack.
```

2. If granted, the authorization server respond with a 302 redirect to the client point with following parameters:

```
HTTP/1.1 302 Found
    Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
            &state=xyz

code MUST include. Length not sure.
state MUST include if client has included.

code only lives for max 10 mins. And can be used only once. Implementation should be strict.
```

3. If the request failed by the request itself, server must not re-direct and inform the user of the error. If the request failed because user denied access granting, or other below listed errors, the redirect still executes, and with the following addtional parameter:

```
HTTP/1.1 302 Found
   Location: https://client.example.com/cb?error=access_denied&state=xyz

state: REQUIRED if client was set by the user
error: invalid_request/unauthorized_client/access_denied/unsupported_response_type/invalid_scope/server_error/temporarily_unavailable

error_description: OPTIONAL, use ASCII codes
```

4. Request Access Token
All parameter must URL encoded. Must use POST to avoid http-forwarded-for leaking. Request to exchange "authorization code" into a usable "access token". Example:

```
POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
     &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

     grant_type: MUST be "authorization_code"
     code: MUST
     redirect_uri: MUST if included in step 1. And must be identical.
     client_id: MUST if dont use Authorization header in HTTP request.
```

Server must:
* Validates the client credentials.
* Validates the client matches the authorization code issued.
* Validates the authorization code.
* Validates that Redirect URI is identical.


5. Access Token Response
If granted issue access token (optionally with a refresh token). If denied return error response.

## Implicit Mode
Mainly used in browser. Since the client (Javascript program) resides in the user-agent (web browser) and can interact with the web browser directly. And in this type no "refresh token" is issued. Access token is directly issued.

```
     +----------+
     | Resource |
     |  Owner   |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier     +---------------+
     |         -+----(A)-- & Redirection URI --->|               |
     |  User-   |                                | Authorization |
     |  Agent  -|----(B)-- User authenticates -->|     Server    |
     |          |                                |               |
     |          |<---(C)--- Redirection URI ----<|               |
     |          |          with Access Token     +---------------+
     |          |            in Fragment
     |          |                                +---------------+
     |          |----(D)--- Redirection URI ---->|   Web-Hosted  |
     |          |          without Fragment      |     Client    |
     |          |                                |    Resource   |
     |     (F)  |<---(E)------- Script ---------<|               |
     |          |                                +---------------+
     +-|--------+
       |    |
      (A)  (G) Access Token
       |    |
       ^    v
     +---------+
     |         |
     |  Client |
     |         |
     +---------+

```

1. Client directs user to authorization server.
```
GET /authorize?
response_type=token
&client_id=s6BhdRkqt3
&state=xyz
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb 
    HTTP/1.1
    Host: server.example.com

response_type: MUST be token
client_id: MUST
redirect_uri: OPTIONAL because it is determined when registration.
scope: OPTIONAL
state: OPTIONAL for client-server to track the converstaion.
```

2. Access Token Response
