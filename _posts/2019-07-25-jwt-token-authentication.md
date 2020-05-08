---
layout: post
title: JWT Token Authentication
subtitle: Application for JWT Token Authentication  in stateless application
comments: true
---
# Reference links
1. [Getting Token Authentication Right in a Stateless Single Page Application](https://medium.com/lightrail/getting-token-authentication-right-in-a-stateless-single-page-application-57d0c6474e3)
2. [Authentication in SPA (ReactJS and VueJS) the right way](https://medium.com/@jcbaey/authentication-in-spa-reactjs-and-vuejs-the-right-way-e4a9ac5cd9a3)
3 .[Same Site Cookie](https://web.dev/samesite-cookies-explained/)
# Components can join in authentication process
## Bearer Token
A bearer token is a value that goes into the Authorization header of any HTTP requests. It is not automatically stored anywhere, it has no expiry date and no associated domain. It’s just a value.
Bearer Token can provided by server 
Client side can access it. 

It is usually used in OAuth2 authentication, when client want to get the security token from 3rd server, they use bearer token provided by authentication server, then send this bearer token along with the credential to 3rd server
[What is OAuth2 bearer toekn exactly](https://stackoverflow.com/questions/25838183/what-is-the-oauth-2-0-bearer-token-exactly/25843058)

## Cookie 
There are 2 kinds of cookies (source):

Session cookies: The cookie is deleted when the client shuts down because it doesn’t specify an Expires or Max-Age directive. However, web browsers may use session restoring, which makes most session cookies permanent, as if the browser was never closed. The session timeout must be handled on the server side.

Permanent cookies
Instead of expiring when the client closes, permanent cookies expire at a specific date (Expires) or after a specific length of time (Max-Age).

Server cookies can be configured with several options:

HttpOnly cookies: Browser javascript cannot read them. 

Secure cookie: the browser includes the cookie in an HTTP request only if the request is transmitted over a secure channel (typically HTTPS).

SameSite cookies let servers require that a cookie shouldn’t be sent with cross-site requests, which somewhat protects against cross-site request forgery attacks (CSRF). SameSite cookies are still experimental and are not supported by all browsers yet.


{: .box-note}
**Note:**  We can prevent XSS attack from stealing the cookie by setting this cookie as HttpOnly Cookie

## JSON Web Token

JWTs take the following form: header.payload.signature 

the header.payload usually in one cookie, the signature in another cookie 

Attributes of header.payload cookie
-Expire time
-Secure

Attribute of signature cookie
-secure
-MUST BE HTTP ONLY

## X-Requested-With header
[What's X-Requested-With-header used for?](https://stackoverflow.com/questions/17478731/whats-the-point-of-the-x-requested-with-header)

{: .box-note}
**Note:**  Can prevent the CRSF attack by do the validation  x-requested-with: XMLHttpRequest in the header

## Authentication Flow

![Authentication](/img/authentication_flow.png)

{: .box-note}
**Note:**  Although attacker can't access to signature cookie, however , if they know how the signature can be generated from the header.payload, they can generate the signature at their side, and setting it as HTTP ONLY cookie, then send the header.payload along with faked header.signature to perform CRSF attrack: https://markitzeroday.com/x-requested-with/cors/2017/06/29/csrf-mitigation-for-ajax-requests.html. So the step check x-requested-with: XMLHttpRequest to see if the request is cross domain is necessary.


## CRSF and JWT Token
[JWT and CRSF protection workflow](https://security.stackexchange.com/questions/168403/jwt-and-csrf-protection-workflow)



