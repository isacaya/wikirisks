<img width="100%" height="100%" alt="Wikirisks" src="./resources/Wikirisks.png" />

- - -

**WikiRisks** is a repository dedicated to organizing web application vulnerabilities with a strong focus on their risks and impacts, not on discovery methodologies or root causes. The project aims to assist those in the security field ensure that the vulnerabilities they discover are properly recognized for their significance.

## Table of Contents

Each document is organized by **impact**: the headings describe what an attacker can achieve, followed by concrete cases that demonstrate it.

| Vulnerability | Impacts |
| --- | --- |
| [Arbitrary Code Execution](./Arbitrary_Code_Execution.md) | Denial of Service, Ransomware Deployment, Cryptojacking |
| [Cache Poisoning](./Cache_Poisoning.md) | Chaining with Denial of Service, Chaining with XSS |
| [Clickjacking](./Clickjacking.md) | Account Takeover, Authorization Grant Hijacking, Bypassing Anti-CSRF Protections |
| [CORS Misconfiguration](./CORS_Misconfiguration.md) | Data Exfiltration, Bypassing IP-based Authentication |
| [Credential Exposure](./Credential_Exposure.md) | Access Token Exposure |
| [CRLF Injection](./CRLF_Injection.md) | XSS, Session Fixation, Cache Poisoning |
| [Cross-Domain Referer Leakage](./Cross-Domain_Referer_Leakage.md) | Cross-Domain Referer Leakage |
| [Cross-Site WebSocket Hijacking](./Cross-Site_WebSocket_Hijacking.md) | Data Exfiltration, Account Takeover, Data Tampering |
| [CSRF (Cross-Site Request Forgery)](./CSRF.md) | Account Takeover, Privilege Escalation, Financial Loss, Data Tampering, Chaining with XSS, Login CSRF |
| [CSS Injection](./CSS_Injection.md) | Data Exfiltration, UI Redressing (Phishing) |
| [Error Message Exposure](./Error_Message_Exposure.md) | Sensitive Data in Error Messages |
| [HTTP Request Smuggling](./HTTP_Request_Smuggling.md) | Access Control Bypass, Capturing Other Users' Requests, Response Queue Poisoning, Chaining with Cache Poisoning, Chaining with XSS |
| [Insecure Deserialization](./Insecure_Deserialization.md) | RCE, DoS, Object Injection |
| [Internal Service Exposure](./Internal_Service_Exposure.md) | Kubernetes API Endpoint Exposure |
| [Lack of Rate Limit](./Lack_of_Ratelimit.md) | Account Takeover, Account Lockout |
| [NoSQL Injection](./NoSQL_Injection.md) | Authentication Bypass, Data Exfiltration, Data Tampering |
| [OAuth Misconfiguration](./OAuth_Misconfiguration.md) | Account Takeover, Data Exfiltration |
| [Open Redirect](./Open_Redirect.md) | Phishing |
| [Path Traversal](./Path_Traversal.md) | Information Disclosure, Arbitrary File Modification |
| [Prototype Pollution](./Prototype_Pollution.md) | RCE, Privilege Escalation, XSS, Denial of Service |
| [Race Condition](./Race_Condition.md) | Limit Overrun, Authentication Bypass |
| [Session Fixation](./Session_Fixation.md) | Session Hijacking |
| [SQL Injection](./SQL_Injection.md) | Information Leakage, Authentication Bypass, RCE |
| [SSRF](./SSRF.md) | Access Control Bypass |
| [Subdomain Takeover](./Subdomain_Takeover.md) | Bypass Browser Security Mechanisms, Phishing with Trusted Domains |
| [Template Injection](./Template_Injection.md) | RCE, Client-Side Protection Bypass |
| [Unrestricted File Upload](./Unrestricted_File_Upload.md) | Upload a Web Shell, Upload a File with an HTML Extension, Distribute Malicious Files |
| [Weak Credential Management](./Weak_Credential_Management.md) | Known or Guessable Secrets, Weak Encryption of Secrets |
| [XSS (Cross-Site Scripting)](./XSS.md) | Credential Stealing, Chaining with CSRF, Web API Abusing, Content Spoofing, Data Exfiltration, Denial of Service |
| [XXE (XML External Entity Injection)](./XXE.md) | DoS, Arbitrary File Read, Reconnaissance, SSRF |
| [Zip Slip](./Zip_Slip.md) | RCE, Persistent Access, Denial of Service |

## Contributing

**WikiRisks** is an open-source project. If you’re interested in contributing, please refer to the [CONTRIBUTING.md](https://github.com/isacaya/wikirisks/blob/main/CONTRIBUTING.md) file for guidelines on how to get started. Your insights are valuable to the project!
