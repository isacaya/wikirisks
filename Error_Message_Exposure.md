# Error Message Exposure

## Table of Contents
- [Sensitive Data in Error Messages](#sensitive-data-in-error-messages)

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