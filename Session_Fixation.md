# Session Fixation

## Table of Contents
- [Session hijacking](#session-hijacking)

## Session hijacking

### Attackers can hijack a user session through fixed session ID.

- Retrieving schema objects
  1. An attacker logs into a vulnerable service on a public computer (e.g., in a library), captures the session information, and then logs out.
  2. Later, a victim uses the same public computer to log into the vulnerable service.
  3. The attacker then uses the session information recorded earlier to hijack the victim’s session.