# Learning Notes

This is where I write down what I learn, in simple words, so I can read it again if I forget and so anyone else can learn the same topic from scratch.

## What we are doing

- Each topic gets its own markdown file.
- Every file explains the idea step by step, with diagrams and small code examples.
- Every file ends with a checklist and common mistakes for quick review.

## Topics

### [Introduction to Backend Development](backend-development-intro.md)

The big picture of backend engineering. Start here. Covered in this note:

- [What backend development is](backend-development-intro.md#1-what-is-backend-development)
- [The request lifecycle](backend-development-intro.md#2-the-request-lifecycle)
- [Languages and frameworks](backend-development-intro.md#4-languages-and-frameworks)
- [APIs (REST, GraphQL, gRPC, WebSockets)](backend-development-intro.md#5-apis)
- [Databases](backend-development-intro.md#6-databases)
- [Caching](backend-development-intro.md#8-caching)
- [Background jobs and queues](backend-development-intro.md#9-background-jobs-and-queues)
- [File storage](backend-development-intro.md#10-file-storage)
- [Application architecture](backend-development-intro.md#11-application-architecture)
- [Concurrency, scalability and performance](backend-development-intro.md#12-concurrency)
- [Reliability and observability](backend-development-intro.md#14-reliability-and-observability)
- [Security](backend-development-intro.md#15-security)
- [Testing](backend-development-intro.md#16-testing)
- [Git, DevOps, and environments](backend-development-intro.md#17-git-and-collaboration)
- [Learning roadmap and project ideas](backend-development-intro.md#20-learning-roadmap)

### [Authentication and Authorization](authn-authz-guide.md)

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

### [HTTP and HTTPS](http-https-guide.md)

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

### [API Design Checklist](api-design-checklist.md)

A short practical reminder for building safer, more predictable backend APIs.

- Clear route design and consistent contracts
- Input validation and error handling
- Idempotency and duplicate-safe actions
- Security, observability, and scaling basics

_More topics will be added here as I learn them._
