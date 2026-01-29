# OWASP Top 10 Reference

Quick reference for OWASP Top 10 2021 vulnerability categories.

## Categories

### A01: Broken Access Control

Access control enforces policy such that users cannot act outside their intended permissions. Failures typically lead to unauthorized information disclosure, modification, or destruction of data, or performing business functions outside the user's limits.

**Common patterns:** Missing authorization checks, insecure direct object references (IDOR), directory traversal, accessing API with missing access controls, elevation of privilege.

### A02: Cryptographic Failures

Failures related to cryptography which often lead to sensitive data exposure. This includes using weak or obsolete cryptographic algorithms, improper key management, and transmitting data in clear text.

**Common patterns:** Weak hashing algorithms (MD5, SHA1 for passwords), missing encryption for sensitive data, hardcoded encryption keys, using deprecated TLS versions.

### A03: Injection

Injection flaws occur when untrusted data is sent to an interpreter as part of a command or query. The attacker's hostile data can trick the interpreter into executing unintended commands or accessing unauthorized data.

**Common patterns:** SQL concatenation, `eval()`, `exec()`, template injection, LDAP injection, OS command injection, XPath injection.

### A04: Insecure Design

Insecure design refers to risks related to design and architectural flaws. This category focuses on risks related to design flaws, calling for more use of threat modeling, secure design patterns, and reference architectures.

**Common patterns:** Missing rate limiting, business logic flaws, lack of input validation at design level, missing multi-factor authentication for critical functions.

### A05: Security Misconfiguration

The application might be vulnerable if it is missing appropriate security hardening, has unnecessary features enabled, uses default accounts/passwords, or reveals overly detailed error messages.

**Common patterns:** Debug mode enabled in production, default credentials, verbose error messages exposing stack traces, unnecessary services running, missing security headers.

### A06: Vulnerable and Outdated Components

Components such as libraries, frameworks, and software modules run with the same privileges as the application. If a vulnerable component is exploited, it can result in serious data loss or server takeover.

**Common patterns:** Outdated dependencies with known CVEs, unmaintained packages, components without security patches. Check with `npm audit`, `dotnet list package --vulnerable`, `pip-audit`.

### A07: Identification and Authentication Failures

Confirmation of the user's identity, authentication, and session management is critical to protect against authentication-related attacks. Weaknesses in these areas can lead to account compromise.

**Common patterns:** Missing session validation, weak password requirements, credential stuffing vulnerabilities, session fixation, missing account lockout, exposing session IDs in URLs.

### A08: Software and Data Integrity Failures

Software and data integrity failures relate to code and infrastructure that does not protect against integrity violations. This includes insecure deserialization, using software from untrusted sources, and CI/CD pipeline vulnerabilities.

**Common patterns:** Unsafe deserialization, unsigned updates, untrusted CDN sources, CI/CD pipeline without integrity verification, auto-update without signature validation.

### A09: Security Logging and Monitoring Failures

This category helps detect, escalate, and respond to active breaches. Without logging and monitoring, breaches cannot be detected. Insufficient logging, detection, monitoring, and active response occurs commonly.

**Common patterns:** Missing audit logs for sensitive operations, logging sensitive data (passwords, tokens), logs not monitored for suspicious activity, no alerting on security events.

### A10: Server-Side Request Forgery (SSRF)

SSRF flaws occur when a web application fetches a remote resource without validating the user-supplied URL. It allows attackers to coerce the application to send crafted requests to unexpected destinations.

**Common patterns:** Unvalidated URL fetching, internal service access via URL manipulation, cloud metadata endpoint access, port scanning via SSRF, protocol smuggling.

## Quick Detection Patterns

| Category | Key Patterns to Search For |
|----------|---------------------------|
| A01 | Missing authz checks, IDOR, directory traversal, `../`, role checks |
| A02 | MD5, SHA1, hardcoded keys, `password`, `secret`, `apiKey` in code |
| A03 | String concatenation in queries, `eval(`, `exec(`, `${}` in SQL |
| A04 | Missing rate limits, no CAPTCHA, business logic without validation |
| A05 | `DEBUG=true`, default ports, verbose errors, missing HTTPS |
| A06 | Old package versions, `npm audit`, `dotnet list`, outdated deps |
| A07 | Weak password regex, missing MFA, session in URL, no lockout |
| A08 | `deserialize(`, `pickle.loads(`, `unserialize(`, unsigned data |
| A09 | Missing `log.`, sensitive data in logs, no audit trail |
| A10 | `fetch(userInput)`, `request(url)`, `curl`, internal IPs |
