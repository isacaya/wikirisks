# Weak Credential Management

## Table of Contents
- [Known or Guessable Secrets](#known-or-guessable-secrets)
- [Weak Encryption of Secrets](#weak-encryption-of-secrets)

## Known or Guessable Secrets

### Using secrets that are easily guessed or commonly found in wordlists can compromise the service.

- JWT (HS256) secret key crack
   ```bash
   hashcat -m 16500 -a 0 hash.txt secret_key_list.txt
   ```
   
- Easily guessable admin credentials
   ```bash
   curl -X POST -d "username=admin&password=admin" https://[VULNERABLE-SERVICE]/login
   ```

## Weak Encryption of Secrets

### Using weak encryption algorithms enables faster decryption, potentially leading to further attacks. (e.g., credential stuffing)

- Cracking MD5 or SHA-1 hashed passwords
   ```bash
   hashcat -m 0 -a 0 hash.txt wordlist.txt
   ```
