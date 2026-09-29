# Subdomain Takeover

## Table of Contents
- [Bypass Browser Security Mechanisms](#bypass-browser-security-mechanisms)
- [Phishing with Trusted Domains](#phishing-with-trusted-domains)

## Bypass Browser Security Mechanisms

### A compromised subdomain can be used to bypass the SameSite.

- Even if SameSite is applied to mitigate CSRF attacks, in most cases, attackers can still exploit CSRF.

### Data leakage can occur even with relatively secure CORS configurations.

- Hackerone report, [Subdomain Takeover to Authentication bypass](https://hackerone.com/reports/335330)

## Phishing with Trusted Domains

### Phishing can be effectively carried out through trusted domain.
