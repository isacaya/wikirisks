# Race Condition

## Table of Contents
- [Limit Overrun](#limit-overrun)
- [Authentication Bypass](#authentication-bypass)

## Limit Overrun

### The count limit can be bypassed.

- Reusing a gift card multiple times
    - Hackerone report, [Race Condition allows to redeem multiple times gift cards which leads to free "money"](https://hackerone.com/reports/759247)

- Receive one-time credits multiple times
    - Hackerone report, [Race Condition in account survey](https://hackerone.com/reports/165570)

## Authentication Bypass

### Authentication can be bypassed.

- Bypassing email-based authentication
    - PortSwigger's research (by albinowax), [Discovering a race condition vulnerability in Gitlab with the single-packet attack](https://www.youtube.com/watch?v=Y0NVIVucQNE)
