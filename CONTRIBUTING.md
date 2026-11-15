# Contributing to WikiRisks
Thank you for your interest in contributing to **WikiRisks**! Contributions are welcome from everyone who wants to help improve the project. Here’s how to get involved:

## How to Contribute
1. Fork the repository.
2. Make your changes in a new branch.
3. Create a pull request with a brief description of your changes.

## Ways to Contribute
### Add new vulnerability or risk of vulnerability example
- Contribute by adding new vulnerability types, attack vectors, or examples.

### Improve documentation by updating content
- Improve existing vulnerability descriptions, clarify the impact, or provide new references.

### Fix errors
- Report or fix typographical or factual errors in vulnerability details or examples.

### ETC...
- Any other contributions that can help improve the project are welcome! Whether it's suggestions, new ideas, or fixes, feel free to contribute.

## Vulnerability Template

The template is as follows.

````markdown
# Vulnerability Title

## Table of Contents

- [Short Description of the Impact](#short-description-of-the-impact)

## Short Description of the Impact

### One sentence describing what the attacker can achieve. [^1]

- Case 1
    1. Step or context (e.g., the vulnerable code or request)
        ```language
        [Code snippet or request showing the impact]
        ```

    2. Step or result (e.g., the manipulated code or resulting response)
        ```language
        [Code snippet or response showing the impact]
        ```

- Case 2
    - Reference title, [Report or write-up title](https://example.com/report)

[^1]: Use footnotes when there are specific requirements or constraints for the vulnerability.
````

If a case needs an image, upload it to the `resources` folder. Name the file by combining the section title and the case name, with spaces replaced by underscores (e.g., `Upload_a_file_with_an_HTML_extension_phishing.png`).

## Writing Guidelines

### Focus on impact
- Organize each document by **what an attacker can achieve** (e.g., Account Takeover, Data Exfiltration, RCE), not by how the vulnerability is found or why it occurs.
- Reuse existing impact names (e.g., `Account Takeover`, `Data Exfiltration`, `Denial of Service`) when they fit, so documents stay consistent.
- When a vulnerability becomes more dangerous in combination with another one (e.g., a self-targeted XSS escalated through cache poisoning), describe it in a `Chaining with ...` section.

### Structure
- Name the file after the title, replacing spaces with underscores (e.g., `Open_Redirect.md`).
- Use `#` for the vulnerability title, `##` for each impact, and `###` for the one-sentence description of that impact. Write headings in Title Case; the `###` description is a sentence and ends with a period.
- Every `##` impact section must have a matching entry in the Table of Contents. Table of Contents links use the GitHub-style anchor: lowercase, spaces replaced with hyphens (`#short-description-of-the-impact`), not underscores.
- Add a new document to the table in [README.md](./README.md).
- Indent nested content with four spaces, and end every file with a newline and no trailing whitespace.

### Examples
- Prefer a minimal inline example (code, request, or response) that shows the impact. Use `1.` / `2.` steps when the vulnerable code and the manipulated result both help the reader.
- Add a link to a public report or write-up as a reference, not as the only content, because external links can disappear. Use a descriptive title, as in `Hackerone report, [Title](URL)`.
- Specify the language on every code block (e.g., `html`, `http`, `bash`).

### Placeholders and sensitive information
- Never use real domains, IP addresses, credentials, tokens, or personal data. Use the placeholders below:
  - `[VULNERABLE-SERVICE]`: the vulnerable application
  - `[ATTACKER-DOMAIN]`: a domain controlled by the attacker
  - `[IDENTITY-PROVIDER]`: an OAuth or SSO provider
  - Other bracketed uppercase names (e.g., `[ATTACKER-CLIENT-ID]`) follow the same pattern.

### Footnotes and prerequisites
- When an impact only applies under specific conditions (configuration, privileges, browser behavior), state them in a footnote, such as "Applies when the session cookie is set with `SameSite=None`."
- Footnote numbers restart in each document. Every `[^n]` marker must have a matching `[^n]:` definition.
