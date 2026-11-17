# Cross-Domain Referer Leakage

## Table of Contents

- [Information Disclosure](#information-disclosure)

## Information Disclosure

### Sensitive information included in the URI may be exposed during cross-domain requests.

- Cross-domain referer leakage [^1]
    - HackerOne report, [Password reset token leakage via referer](https://hackerone.com/reports/342693)

[^1]: Applies when the page's `Referrer-Policy` allows the full URL to be sent to other origins (e.g., `unsafe-url`). See MDN, [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Referrer-Policy#unsafe-url_2).
