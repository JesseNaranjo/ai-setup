# OWASP Top 10 Quick Reference

| OWASP Category | Key Patterns |
|----------------|--------------|
| A01 Broken Access Control | Missing authz checks, IDOR, directory traversal |
| A02 Cryptographic Failures | Weak hashing, missing encryption, hardcoded keys |
| A03 Injection | SQL concat, eval(), exec(), template injection |
| A04 Insecure Design | Missing rate limits, business logic flaws |
| A05 Security Misconfiguration | Debug enabled, default credentials, verbose errors |
| A06 Vulnerable Components | Outdated dependencies (check with npm audit, dotnet list) |
| A07 Auth Failures | Missing session validation, weak passwords |
| A08 Data Integrity Failures | Unsafe deserialization, unsigned data |
| A09 Logging Failures | Missing audit logs, logging sensitive data |
| A10 SSRF | Unvalidated URL fetching, internal service access |
