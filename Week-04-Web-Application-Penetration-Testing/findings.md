# Findings – Web Application Penetration Testing (OWASP Top 10)

## Executive Summary

This assessment covered two intentionally vulnerable web applications: OWASP Mutillidae II (hosted on Metasploitable 2) and DVWA (deployed via Docker). Testing was performed in an isolated lab environment against the OWASP Top 10 categories. Eight vulnerabilities were confirmed and exploited, and three controls were tested and found effective.

The most significant results were a SQL injection vulnerability that allowed full extraction of the application database, including plaintext account credentials, and an OS command injection vulnerability that allowed command execution in the web server's privilege context. Both are rated Critical. Several additional issues compounded the risk: the application database account was over-privileged, the stored credentials were unhashed, and the underlying software stack (operating system, PHP, MySQL, Apache) was long past end-of-life.

---

# Assessment Statistics

| Metric | Result |
|---------|-------:|
| Confirmed vulnerabilities | 8 |
| Critical severity | 2 |
| High severity | 3 |
| Medium severity | 2 |
| Negative findings (controls verified effective) | 3 |
| Tasks completed | 4 |
| Databases enumerated via SQL injection | 7 |
| Database tables enumerated in the application database | 6 |
| Account records extracted | 19 (contents redacted) |
| Requests used for SQLMap injection confirmation | 290 |

---

# Confirmed Vulnerabilities

| # | Vulnerability | Location | OWASP Category | Severity |
|---|---------------|----------|----------------|----------|
| 1 | SQL Injection (manual and SQLMap) | User-info page, username parameter (Mutillidae) | A03:2021 Injection | Critical |
| 2 | Plaintext password storage | Application account table (Mutillidae database) | A02:2021 Cryptographic Failures | Critical |
| 3 | OS Command Injection | DVWA Command Injection module | A03:2021 Injection | Critical |
| 4 | Insecure Direct Object Reference (IDOR) | Blog-viewing page, author parameter (Mutillidae) | A01:2021 Broken Access Control | High |
| 5 | Reflected Cross-Site Scripting (XSS) | DNS Lookup tool (Mutillidae) | A03:2021 Injection | High |
| 6 | Local File Inclusion (LFI) | Page parameter, application-wide (Mutillidae) | A03:2021 Injection | High |
| 7 | Excessive database privileges | Application database connection (Mutillidae) | A05:2021 Security Misconfiguration | Medium |
| 8 | Default credentials accepted without lockout | Mutillidae login | A07:2021 Identification and Authentication Failures | Medium |

---

# Tested, Not Vulnerable (Negative Findings)

| Attack Vector | Location | Result |
|---------------|----------|--------|
| Username-substitution IDOR | User-info page (Mutillidae) | Username and password validated as a matched pair; substitution rejected |
| Cookie-based privilege escalation | Application-wide (Mutillidae) | No client-exposed role cookie or hidden field; authorization kept server-side in the PHP session |
| Remote File Inclusion (RFI) | Page parameter (Mutillidae) | Blocked by PHP's `allow_url_include=Off` setting |

---

# Security Observations

## Injection Vulnerabilities (SQL Injection, OS Command Injection, LFI)

Three distinct injection classes were confirmed. The SQL injection flaw allowed the full application database to be enumerated and extracted. The OS command injection flaw allowed arbitrary commands to run as the web server user. The LFI flaw allowed reading of arbitrary files readable by the web server process. In each case, user input reached a sensitive sink (a database query, a shell command, a file path) without validation or parameterization. The common fix is parameterized queries, avoiding shell calls with user input, and strict allow-listing of file paths.

## Credential Storage

Passwords were stored in plaintext. This is a standalone finding: even without the SQL injection flaw, any read access to the database would expose every user's password. Passwords should be hashed with a slow, salted algorithm such as bcrypt, scrypt, or Argon2.

## Authentication and Access Control

The application accepted a default administrator credential pair with no lockout or rate-limiting, which allowed automated credential guessing to succeed. An IDOR flaw on the blog page allowed any authenticated user to read another user's private entries by changing a single parameter. Authorization must be enforced server-side on every object access, not assumed from the user's session alone.

## Client-Side Execution

Reflected input was rendered without output encoding, allowing script execution in the victim's browser. Session cookies were readable by JavaScript, so a successful injection could hijack a user session. Output encoding for the rendering context and the HttpOnly cookie attribute are the standard mitigations.

## Infrastructure and Configuration

The database connection ran with excessive privileges, and the underlying operating system, web server, PHP, and database versions were long past end-of-life with known unpatched vulnerabilities. Lab targets are expected to be outdated, but the same pattern in production would represent a major exposure. The `allow_url_include=Off` setting is a positive control and should remain disabled.

---

# Recommendations for Future Assessment

1. Test every endpoint for authorization independently, rather than validating one page and assuming the rest of the application behaves the same way.
2. Include negative results in every report, with the specific control that blocked the attack, so that effective defenses are documented alongside weaknesses.
3. Retest injection points with small syntax variations (spacing, separators, comment styles) before concluding a target is not vulnerable.
4. Verify session state and re-validate baseline behavior after any VM reboot or environment change before interpreting failed payloads.
5. Map every finding to an OWASP category and a severity rating, and state the impact in terms of what an attacker could do with it.
6. Prioritize credential and data-exposure findings in the report, since they enable further attacks beyond the initial vulnerability.

---

# Conclusion

Week 4 covered the core OWASP Top 10 web attack surface through Burp Suite traffic manipulation, manual and automated SQL injection, OS command injection, broken access control, reflected XSS, and file inclusion. Testing confirmed eight exploitable vulnerabilities, including two Critical findings, and three negative results showing that specific controls worked as intended.

The most important result is the combination of injection and credential storage weaknesses, which together would allow a complete compromise of the application's user accounts. The negative findings are equally important: they show that credential pairing, server-side session state, and PHP's file inclusion hardening were functioning correctly. Together, these results form a complete and honest picture of the target's security posture.

---
