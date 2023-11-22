---
layout: post
title: "An introduction of OAuth"
date: 2023-11-22 12:00:00
updated: 2023-11-22 12:00:00
copyright: true
categories:
  - Security
tags:
  - Security
  - OAuth
---
# Overview

## What is OAuth 2.0？

> [RFC Reader - An online reader for IETF RFCs](http://www.rfcreader.com/#rfc6749)

The OAuth (Open Authorization) 2.0 authorization framework enables a third-party application to obtain limited access to an HTTP service, either on behalf of a resource owner by orchestrating an approval interaction between the resource owner and the HTTP service, or by allowing the third-party application to obtain access on its own behalf. This specification replaces and obsoletes the OAuth 1.0 protocol described in RFC 5849.
<!-- more -->

## Analogy

### Scenario

I live in a large residential community with a gated access system that requires a password for entry.

As a frequent online shopper and food delivery recipient, I receive daily visits from delivery personnel.

I need to find a solution to allow these delivery personnel to access the community through the gate without having to share my personal password.

If I were to share my password with them, they would have the same level of access as I do, which doesn't seem appropriate. Additionally, it would be troublesome if I ever wanted to revoke their access, as I would need to change my password and inform other delivery personnel as well.

Is there a way to grant the delivery personnel the freedom to enter the community for deliveries without them knowing the my passwords, while restricting their access to only delivery-related activities and preventing access to other secured areas that require a password?

### Authorization mechanism

As a result, I have devised an authorization mechanism:

Step 1: Under the password input panel of the access control system, add a button called "Request Authorization." Delivery personnel need to press this button to apply for access authorization.

Step 2: After pressing the button, a dialogue box will appear on the my phone, indicating that someone is requesting authorization. The system will also display the name, employee ID, and the delivery company the personnel belongs to.

Upon verifying the request, I can click the button to inform the access control system that I agree to grant them access to the community.

Step 3: Once the access control system receives my confirmation, it will display an access token to the delivery personnel. The token is a unique set of numbers similar to a password, valid for a short period (e.g., seven days).

Step 4: The delivery personnel will input the access token into the access control system to enter the community.

Some may wonder why not remotely open the door for the delivery personnel instead of generating a unique access token for them. The reason for this is that delivery personnel may come to make deliveries daily, and the next day, they can reuse the same access token.

### Difference between token and password

(1) Tokens are `short-term` and expire automatically, whereas passwords tend to be `long-term` unless actively changed by the user.

(2) Tokens `can be revoked` by the data owner `at any time`, immediately becoming invalid. In the example above, the homeowner could revoke the courier's token access. Passwords generally cannot be rescinded by others.

(3) Tokens have `scoped permissions`, such as only granting access through Gate. For web services, a read-only token is more secure than read/write. Passwords usually convey full permissions.

In summary, tokens and passwords serve similar roles in granting system access, but tokens provide more granular control and can be revoked remotely.

## Terms

|name|description|
|-|-|
|Third-Party Application|Obtains user information. The delivery personnel acts as a third-party application and seeks to pass through the access control system to enter the residential community.
|Resource Owner|The individual who owns or possesses the resources, and in this context, it refers to the user. The term "resources" denotes various URL interfaces and data requested.
|Authorization Server|The server responsible for authentication, specifically designated by the service provider (e.g., the community) for handling the authentication process.
|Resource Server|The server where the service provider stores user resources. It can be the same server as the authorization server or a different one.
|Access Token|A token granted to the client for accessing resources from the Resource Server (API). Access tokens are typically short-lived and have a limited duration.

# Authorization flow

![OAuth workflow](/uploads/in-post/security/OAuth-workflow.png)

The workflow of OAuth 2.0 is shown in the following diagram, excerpted from RFC 6749.

![OAuth workflow table](/uploads/in-post/security/OAuth-workflow-table.png)

(A) After the user opens the client application, the client requests authorization from the user.

(B) The user agrees to grant authorization to the client.

(C) The client utilizes the obtained authorization to apply for an access token from the authorization server.

(D) After authenticating the client, the authorization server confirms the accuracy and grants the access token.

(E) The client employs the access token to request resources from the resource server.

(F) The resource server verifies the access token's validity and grants access to the resources to the client.

# Authorization mode

There are four types of authorization modes.
1. Authorization code
2. Implicit
3. Resource owner password credentials
4. Client credentials

## Authorization code

### Overview

***The authorization code grant involves the client requesting an authorization code first, then using that code to obtain a token***.

This is the most common and secure workflow, suitable for web apps that have a backend. The authorization code is passed via the frontend, while the token stored in the backend. All communication with the resource server also happens in the backend. This separation of frontend and backend prevents the token from leaking.

### Workflow

![authorization code workflow table](/uploads/in-post/security/authorization-code-workflow-table.png)

(A) The user visits the client, which then redirects the user to the authorization server.

(B) The user decides whether to grant authorization to the client.

(C) Assuming authorization is granted, the authorization server redirects the user back to the client's pre-specified "redirection URI", along with an authorization code.

(D) The client receives the authorization code, and sends it along with the original "redirection URI" to the authorization server to request a token. This step happens on the backend server of the client, invisible to the user.

(E) The authorization server verifies the authorization code and redirection URI, and if valid, sends an access token and refresh token to the client.

### Example

Website A want to get authorization from Website B

#### Step 1

Website A provides a link which, when clicked by the user, redirects them to Website B to authorize Website A to access the user's data. Below is an example redirect link from Website A to Website B:

```text
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

In the above URL, the `response_type` parameter indicates a request for an authorization code (`code`). The `client_id` parameter informs Website B of the identity of the requesting party. The `redirect_uri` parameter specifies where Website B should redirect after accepting or denying the request. Finally, the `scope` parameter indicates the requested scope of authorization (in this case, read-only access).

![get authorization code](/uploads/in-post/security/get-authorization-code.png)

#### Step 2

After the user is redirected, Website B prompts the user to log in and then asks if they consent to authorize Website A. The user agrees, at which point Website B redirects back to the URL specified in the `redirect_uri` parameter, passing back an authorization code, as follows:

```text
https://a.com/callback?code=AUTHORIZATION_CODE
```

In the URL above, the code parameter represents the authorization code.
![return authorization code](/uploads/in-post/security/return-authorization-code.png)

#### Step 3

Once Website A has obtained the authorization code, it can then make a request to Website B's backend for a token.

```text
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

In the URL above, the `client_id` and `client_secret` parameters allow Website B to verify Website A's identity (the `client_secret` parameter is confidential and can only be used in backend requests), the `grant_type` parameter with a value of `AUTHORIZATION_CODE` indicates that authorization is via an authorization code, the `code` parameter is the authorization code obtained in the previous step, and the `redirect_uri` parameter is the callback URL after the token is issued.

![get authorization token](/uploads/in-post/security/get-authorization-token.png)

#### Step 4
After receiving the request, Website B issues a token. Specifically, it sends a JSON payload to the URL specified in `redirect_uri`.

```text
{
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

In the JSON payload above, the `access_token` field contains the issued token that Website A receives in its backend.

![return authorization token](/uploads/in-post/security/return-authorization-token.png)

## Implicit

### Overview

Some web applications are purely frontend without a backend. In these cases, the above method cannot be used and the token must be stored in the frontend. ***RFC 6749 specifies a second approach that allows tokens to be issued directly to the frontend. Since this method lacks the intermediate authorization code step, it is referred to as the (authorization code) "implicit" flow.***

### Workflow

![implicit workflow table](/uploads/in-post/security/implicit-workflow-table.png)

(A) The client directs the user to the authorization server.

(B) The user decides whether to grant the client authorization.

(C) Assuming authorization is granted, the authorization server redirects the user to the "redirection URI" specified by the client, with an access token embedded in the Hash portion of the URI.

(D) The browser makes a request to the resource server without including the Hash value received in the previous step.

(E) The resource server returns a web page containing code that can extract the token from the Hash value.

(F) The browser executes the script obtained in the previous step to retrieve the token from the Hash.

(G) The browser passes the token to the client.

### Example

Website A want to get authorization from Website B

#### Step 1

Website A provides a link asking the user to redirect to Website B to authorize Website A to access the user's data.

```text
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

In the URL above, the `response_type` parameter with a value of `token` indicates a request to return the token directly.

#### Step 2

The user is redirected to Website B and after logging in, consents to authorize Website A. Website B then redirects back to the URL specified in the `redirect_uri` parameter, passing the token to Website A as a URL parameter.

```text
https://a.com/callback#token=ACCESS_TOKEN
```

In the URL above, the `token` parameter represents the token that Website A receives directly in its frontend.

Note that the token is placed in the URL fragment instead of the query string because OAuth 2.0 allows redirect URIs using the HTTP protocol, which carries the risk of a "man-in-the-middle" attack. By placing the token in the fragment, it is not sent to the server on redirect, reducing the risk of token exposure.

![implicit workflow](/uploads/in-post/security/implicit-workflow.png)

This approach of directly transmitting tokens to the frontend is highly insecure. Therefore, it should only be used in scenarios with low security requirements. The token lifetime must be extremely short, usually valid only for the duration of a session. The token becomes invalid when the browser is closed.

## Resource owner password credentials

### Overview

***RFC 6749 also allows users to directly provide their username and password to highly trusted applications. The application then uses those credentials to obtain a token. This approach is referred to as the "password" flow.***

### Workflow

![password credentials workflow](/uploads/in-post/security/password-credentials-workflow.png)

(A) The user provides their username and password to the client.

(B) The client sends the username and password to the authorization server and requests a token.

(C) After validating the credentials, the authorization server returns an access token to the client.

### Example

Website A want to get authorization from Website B

#### Step 1

Website A asks the user to provide their username and password for Website B. After obtaining the credentials, Website A makes a direct request to Website B for a token.

```text
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

In the above URL, the `grant_type` parameter indicates the authorization method. `password` signifies the "password" flow, while `username` and `password` provide the credentials for Website B.

#### Step 2

After Website B verifies the credentials, it directly returns a token without a redirect. Instead, the token is included in a JSON payload as part of the HTTP response for Website A to retrieve.

This approach requires users to disclose their username/password, which clearly poses significant risks. As such, it should only be used when other authorization methods are unavailable and limited to highly trusted applications.

## Client credentials

### Overview

The client credentials grant refers to the client authenticating directly with the service provider on its own behalf, rather than on behalf of a user. Strictly speaking, this is outside the scope of authorization that OAuth is designed to provide. With this grant type, users register directly with the client, which then requests services from the service provider under its own credentials, rather than via delegation through authorization.

### Workflow

![client credentials workflow](/uploads/in-post/security/client-credentials-workflow.png)

(A) The client authenticates with the authorization server and requests an access token.

(B) After validating the credentials, the authorization server returns an access token to the client.

### Example

Website A want to get authorization from Website B

#### Step 1

Application A makes a request to B through the command line.

```text
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

In the URL above, the grant_type parameter set to client_credentials signifies the client credentials flow is in use. The client_id and client_secret parameters allow Application B to verify Application A's identity.

#### Step 2

After verifying the credentials, Website B directly returns a token.

The token provided through this method pertains to the client application rather than a user. This means the same token may be shared across multiple users.

## Refresh token

Once a token expires, forcing the user to repeat the entire flow to acquire a new one would likely create a poor experience and is unnecessary. OAuth 2.0 allows clients to automatically refresh tokens.

The approach is for Website B to issue two tokens at the time of authorization - one for data access and a second to refresh it (the refresh_token field). Before expiration, the client uses the refresh token to request an updated access token.

```text
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

In the URL above, the `grant_type` parameter set to `refresh_token` signifies this is a request to refresh the token. The `client_id` and `client_secret` parameters are used to verify identity, while the `refresh_token` parameter provides the token meant for this purpose.

After Website B confirms the request is valid, it issues a new access token.
