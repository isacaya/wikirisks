# Open Redirect

## Table of Contents

- [Phishing](#phishing)

## Phishing

### Users can be redirected to malicious sites, leading to phishing.

- Redirect to malicious domain
    ```text
    https://[VULNERABLE-SERVICE]/login?redirect_url=https%3A%2F%2F[ATTACKER-DOMAIN]%2F
    ```
