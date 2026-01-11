# Information Disclosure

## Table of Contents
- [Credential Exposure](#credential-exposure)
- [Sensitive Data in Error Messages](#sensitive-data-in-error-messages)

## Credential Exposure

### Exposed credentials (e.g., access tokens, API keys, etc.) can lead to the compromise of the entire service.

- Github access token exposure
    - Hackerone report, [Github access token exposure](https://hackerone.com/reports/1087489)

- Exposed Kubernetes API Endpoint without authorization
    - Hackerone report, [Exposed Kubernetes API - RCE/Exposed Creds](https://hackerone.com/reports/455645)

- Username and password exposed on GitHub
    - Hackerone report, [Leaked JFrog Artifactory username and password exposed on GitHub](https://hackerone.com/reports/911606)

- Cross-domain referer leakage
    - Hackerone report, [Password reset token leakage via referer](https://hackerone.com/reports/342693)

- Exposure in Version Control History
    ```bash
    git show a1b2c3
    ```

## Sensitive Data in Error Messages

### Detailed error messages may expose sensitive information.

- Database queries exposure
    ```bash
    SQLSTATE[42000]: Syntax error or access violation: 1064 
    You have an error in your SQL syntax near 'SELECT * FROM users WHERE id=1\'' at line 1
    ```

- Exposing sensitive information through stack traces
    ```bash
    requests.exceptions.ConnectionError: HTTPConnectionPool(host='invalid_host', port=8080): Max retries exceeded with url: /data?apikey=secret_hardcoded_key&keyword=test (Caused by NameResolutionError("<urllib3.connection.HTTPConnection object at 0x100000000>: Failed to resolve 'invalid_host' ([Errno 8] nodename nor servname provided, or not known)"))
    ```
