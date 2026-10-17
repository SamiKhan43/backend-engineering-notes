# Learning Notes

This is where I write down what I learn, in simple words, so I can read it again if I forget and so anyone else can learn the same topic from scratch.

## What we are doing

- Each topic gets its own markdown file.
- Every file explains the idea step by step, with diagrams and small code examples.
- Every file ends with a checklist and common mistakes for quick review.

## Topics

### [<span style="color: #6366F1;">Authentication and Authorization</span>](authn-authz-guide.md)

Who you are (authentication) and what you are allowed to do (authorization). Covered in this note:

- [Big picture: AuthN vs session vs AuthZ](authentication-guide.md#1-big-picture)
- [Sessions (server-side)](authentication-guide.md#3-sessions-server-side)
- [JWT (token-based)](authentication-guide.md#4-jwt-token-based)
- [Cookies and their security attributes](authentication-guide.md#5-cookies)
- [Stateful vs stateless](authentication-guide.md#6-stateful-vs-stateless)
- [OAuth 2.0](authentication-guide.md#7-oauth-20)
- [OpenID Connect (OIDC)](authentication-guide.md#8-openid-connect-oidc)
- [Authorization: ownership checks and RBAC](authentication-guide.md#9-authorization-ownership-and-rbac)
- [Error messages and timing attacks](authentication-guide.md#10-error-messages-and-timing-attacks)
- [Practical FastAPI example](authentication-guide.md#12-practical-fastapi-example)
- [Checklist and common pitfalls](authentication-guide.md#14-checklist-10-things-to-remember)

### <span style="color: #BF616A;">[HTTP and HTTPS](http-https-guide.md)</span>

How the web talks: messages, headers, methods, CORS, status codes, caching, and TLS. Covered in this note:

- [HTTP basics and evolution (1.0 to 3)](http-https-guide.md#3-evolution-of-http)
- [HTTP messages](http-https-guide.md#4-http-messages)
- [HTTP headers: why and which types](http-https-guide.md#5-http-headers)
- [HTTP methods](http-https-guide.md#6-http-methods)
- [Idempotent vs non-idempotent](http-https-guide.md#7-idempotent-vs-non-idempotent)
- [OPTIONS method and CORS](http-https-guide.md#8-options-method-and-cors)
- [CORS demo with curl and Burp Suite](http-https-guide.md#9-cors-demo-with-curl-and-burp-suite)
- [Response status codes and demo](http-https-guide.md#10-response-status-codes)
- [HTTP caching](http-https-guide.md#12-http-caching)
- [Content negotiation](http-https-guide.md#13-content-negotiation)
- [Compression](http-https-guide.md#14-compression)
- [Persistent connections and keep-alive](http-https-guide.md#15-persistent-connections-and-keep-alive)
- [Multipart data and chunked transfer](http-https-guide.md#16-multipart-data-and-chunked-transfer)
- [SSL, TLS and HTTPS](http-https-guide.md#17-ssl-tls-and-https)
- [Security headers and what happens when you type a URL](http-https-guide.md#18-security-headers-extra)

_More topics will be added here as I learn them._
