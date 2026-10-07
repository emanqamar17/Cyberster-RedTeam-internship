# Methodology – Web Application Penetration Testing (OWASP Top 10)

## Overview

This week's engagement followed a structured, progression-based methodology designed to build understanding from foundational proxy concepts through advanced injection and access control exploitation. Testing moved from request-level manipulation (Burp Suite mastery) through injection attacks (SQL and OS command) to authorization bypass (IDOR) and client-side/server-side code execution (XSS, LFI, RFI). Each phase built on previous findings and tool familiarity, reinforcing the principle that real penetration testing is iterative—adapting technique to what the target actually exposes rather than forcing pre-planned steps onto an unknown surface.

All testing was conducted against intentionally vulnerable platforms (Metasploitable 2 with Mutillidae II, DVWA) in an isolated, authorized lab environment. Infrastructure was validated before each phase, and session state was re-confirmed after any system interruption (VM reboots, network changes).

---

# Web Application Penetration Testing Workflow
PHASE 1: Request-Level Mastery (Burp Suite)
↓
PHASE 2: Data-Layer Attacks (SQL Injection)
↓
PHASE 3: System-Level Attacks (OS Command Injection)
↓
PHASE 4: Authorization & Access Control (IDOR, Cookie Manipulation)
↓
PHASE 5: Client-Side & File Handling (XSS, LFI, RFI)
↓
PHASE 6: Consolidated Analysis & Risk Summary

---

# Phase 1 – Burp Suite Mastery: The Interceptor

## Objective

Establish foundational fluency with Burp Suite as an HTTP(S) intercepting proxy, mastering the ability to pause, inspect, and modify live requests before they reach the server. This phase built the core skill required for all downstream exploitation: the realization that every request's line, headers, and body are client-controlled and can be rewritten in transit.

## Proxy Configuration

**Target:** Mutillidae (192.168.56.101)

**Activity:** Started Burp Suite in Temporary Project mode; configured FoxyProxy in Firefox to route traffic through Burp's listener on 127.0.0.1:8080; verified interception by visiting a plain HTTP site with Intercept ON and confirming the request paused before forwarding.

**Why it matters:** Without a working proxy, downstream testing (SQLi, IDOR, XSS payload delivery) has no control mechanism. The proxy is the red team's primary interface to the target.

**Expected Outcome:** Confirmed that every HTTP request and response is visible and editable before the server processes it or the browser renders it.

## Intercepting and Modifying Requests

**Activity:** Escalated from harmless header modification (User-Agent) to intercepting live registration and login POST requests, modifying username and credential values in transit before forwarding.

**Examples:**
- Modified User-Agent to "BurpSuiteTraining" on a GET to /mutillidae/index.php → page loaded normally
- Intercepted registration POST (username=intern1), changed username to intern_test in transit → account created under modified value
- Intercepted login POST for newly registered account, changed credentials in-flight to username=admin&password=admin → application authenticated as admin

**Why it matters:** Demonstrates that servers must never trust client-supplied authentication data without server-side verification. The application accepted admin/admin credentials without any rate-limiting or lockout, a live example of OWASP A07 (Identification and Authentication Failures).

**Expected Outcome:** Confirmed that request modification is the foundational attack surface; all downstream techniques (SQLi, IDOR, XSS) rely on this mechanism.

## Repeater and Intruder

**Activity:** Used Repeater to isolate the effect of single parameter changes (page parameter: invalid vs. valid, response length difference confirmed); used Intruder Sniper attack mode with a 5-entry password list against the login endpoint, identifying the correct password by response length outlier (27,335 bytes vs. ~26,690 baseline).

**Why it matters:** Repeater and Intruder automate the manual testing loop and enable detection mechanisms when HTTP status codes alone don't vary (as was the case here—every attempt returned 200 OK; only response length/content differed).

**Expected Outcome:** Confirmed that response-length-based attacks can succeed where status-code detection fails; this mirrors real credential-stuffing and default-password attacks at scale.

---

# Phase 2 – Injection Attacks: SQL & OS Command

## Objective

Demonstrate that unvalidated input reaching a database query or operating system shell can be manipulated to extract unintended data or execute unintended commands. This phase combined manual technique-building (understanding syntax error flow → boolean-based logic → UNION-based data extraction) with automated tooling (SQLMap) to cover both the methodology and real-world efficiency gains.

## Manual SQL Injection (Mutillidae user-info.php)

**Target:** username parameter on http://192.168.56.101/mutillidae/index.php?page=user-info.php

**Activity:**
1. Identified injection point via normal lookup (username=admin) captured in Burp
2. Confirmed SQL syntax error by appending a single quote (username=admin')
3. Confirmed boolean-based query control: username=admin' AND '1'='1 returned full data; username=admin' AND '1'='2 returned empty response
4. Enumerated column count via sequential ORDER BY payloads (1–5 succeeded; ORDER BY 6 returned "Unknown column '6'" error) → 5-column query confirmed
5. Verified UNION SELECT 1,2,3,4,5-- - reflected literal placeholder values (Username=2, Password=3, Signature=4) → column mapping complete
6. Extracted database name, user, and version in single request: UNION SELECT 1,database(),user(),version(),5-- - → owasp10 / root@localhost / 5.0.51a-3ubuntu5

**Why it matters:** Error-based → Boolean-based → UNION-based progression mirrors the real-world escalation path used in penetration tests and breach post-mortems. Each technique builds on the previous; understanding the manual flow is essential for adapting to defenses when automated tools fail.

**Expected Outcome:** Full query structure reverse-engineered; database identified; authentication credentialing (admin/admin) and data structure confirmed.

## Automated SQL Injection (SQLMap)

**Target:** Same username parameter; full database extraction

**Commands Used:**
```bash
sqlmap -u "http://192.168.56.101/mutillidae/index.php?page=user-info.php&username=admin&password=admin&user-info-php-submit-button=View+Account+Details" \
  --cookie="PHPSESSID=<session>" --dbs

sqlmap ... -D owasp10 --tables
sqlmap ... -D owasp10 -T accounts --columns
sqlmap ... -D owasp10 -T accounts --dump
```

**Results:**
- Confirmed injection via boolean-based blind, error-based (FLOOR), and time-based blind techniques (290 requests total)
- Fingerprinted backend: MySQL ≥ 4.1, Linux Ubuntu 8.04, Apache 2.2.8, PHP 5.2.4
- Enumerated 7 databases (dvwa, information_schema, metasploit, mysql, owasp10, tikiwiki, tikiwiki195)
- Enumerated 6 tables in owasp10 (accounts, blogs_table, captured_data, credit_cards, hitlog, pen_test_tools)
- Extracted 5 columns from accounts table; dumped all 19 rows with plaintext passwords and is_admin flags

**Critical Finding:** All 19 account passwords stored in plaintext—a standalone, severe finding independent of the SQL injection vulnerability itself. Violates OWASP A02:2021 (Cryptographic Failures) and PCI-DSS.

**Why it matters:** SQLMap is standard tooling in real penetration test methodology (PTES, OWASP Testing Guide). Understanding the output (injection techniques, fingerprint accuracy, full data extraction) is essential for production assessments.

**Expected Outcome:** Complete database compromise confirmed; credential set fully extracted; infrastructure weaknesses (old OS, outdated database, root privilege) documented.

## OS Command Injection (DVWA)

**Target:** DVWA Command Injection module; ping IP field

**Activity:**
1. Established baseline with valid input (127.0.0.1) → normal ping output
2. Tested 127.0.0.1 ; whoami (space before separator) → no injected output; payload may not have been parsed
3. Tested 127.0.0.1; whoami (no space) → ping output followed by www-data → command execution confirmed
4. Tested 127.0.0.1; ls → ping output followed by directory listing (help, index.php, source)

**Why it matters:** Small separator/syntax differences can mask real vulnerabilities. The space-before-semicolon variation produced no visible effect, but the space-less version succeeded—a genuine nuance that separates vulnerability confirmation from false negatives.

**Expected Outcome:** Remote command execution confirmed with www-data privilege level; full filesystem access via directory traversal.

---

# Phase 3 – Broken Access Control & IDOR

## Objective

Test whether applications enforce proper authorization when a client-supplied identifier or role indicator is changed, demonstrating that authentication (you are who you claim to be) is distinct from authorization (you can do what you claim to do).

## Insecure Direct Object Reference (IDOR)

**Target:** Mutillidae view-someones-blog.php; author parameter

**Activity:**
1. Located IDOR surface: Mutillidae's "View Someone's Blog" page selects content via an author parameter
2. Captured request for author=adrian while authenticated as intern_test (a different, unrelated user)
3. Modified identifier to author=admin while remaining authenticated as intern_test → admin's blog entries returned without ownership check

**Why it matters:** IDOR is one of the most common bug bounty submission categories precisely because it requires no technical exploit—just parameter substitution against an unverified ownership check. Any authenticated user can access any other user's data.

**Expected Outcome:** Confirmed that access control must be verified per-endpoint, not assumed application-wide. A different page on the same application (user-info.php) properly validated username and password as a matched pair.

## Privilege Escalation via Cookie Manipulation

**Target:** Mutillidae; role/admin fields in cookies or page source

**Activity:**
1. Logged in as non-admin account (intern_test)
2. Inspected Cookie header → only PHPSESSID present; no role cookie
3. Reviewed page source and response body → no hidden role/admin field or JavaScript variable

**Result (Negative Finding):** Mutillidae keeps authorization state entirely server-side in the PHP session and does not expose a role cookie or hidden field for client tampering. This is the correct, secure design pattern—and is documented as a tested-and-not-vulnerable result, not a missed opportunity.

**Why it matters:** Defense-in-depth: even when parameter tampering is successful (IDOR), other endpoints may have proper controls. Testing both working and broken paths is essential for accurate risk assessment.

**Expected Outcome:** Confirmed that privilege escalation via cookie manipulation is not exploitable on this target; server-side session state is functioning correctly.

---

# Phase 4 – Client-Side Code Execution & File Handling

## Objective

Demonstrate how untrusted input reaching an unsafe sink manifests differently depending on where that sink lives: client-side (JavaScript execution → XSS), server-side file operations (LFI/RFI).

## Reflected Cross-Site Scripting (XSS)

**Target:** Mutillidae OWASP Top 10 → XSS → DNS Lookup tool

**Activity:**
1. Submitted payload `<script>alert(document.cookie)</script>` into the hostname/IP field
2. A JavaScript popup executed, displaying the live PHPSESSID value
3. Confirmed that an attacker could replace the benign popup with silent exfiltration code (e.g., fetch() to an attacker-controlled endpoint)

**Why it matters:** XSS is often dismissed as "just a popup" but is actually a full JavaScript execution context in the victim's browser. The same reflection mechanism that enables alerts enables credential theft, session hijacking, and malware delivery.

**Expected Outcome:** Confirmed session hijacking vector; demonstrated that XSS impact goes far beyond visual popups.

## Local File Inclusion (LFI)

**Target:** Mutillidae page parameter; application-wide

**Activity:**
1. Replaced page parameter with absolute path: index.php?page=/etc/passwd
2. Server returned full contents of /etc/passwd, including Metasploitable's default msfadmin account and service accounts (mysql, postgres, www-data)

**Finding:** The page parameter is used application-wide with no allow-list or path validation. This is an application-wide vulnerability, not a single vulnerable page. Exposed service accounts and the msfadmin default account support further attack planning.

**Why it matters:** LFI enables reading of arbitrary server-readable files: application configuration (database credentials, API keys), system files (/etc/passwd, /proc/self/environ), and source code. It is a stepping stone to full server compromise.

**Expected Outcome:** Confirmed arbitrary file read capability; identified exposed accounts that could be targets for brute-force or escalation.

## Remote File Inclusion (RFI)

**Target:** Same page parameter; remote URL attempt

**Activity:**
1. Hosted a test file (test.txt with content "RFI_TEST") via python3 -m http.server 8000 on Kali
2. Pointed page parameter at the hosted file: page=http://192.168.56.102:8000/test.txt
3. Server returned PHP warning: "URL file-access is disabled in the server configuration"

**Result (Negative Finding):** RFI is not exploitable because PHP's allow_url_include directive is disabled. The application has a dangerous LFI vulnerability, but infrastructure-level hardening (allow_url_include=Off) blocks the more severe RFI half.

**Why it matters:** This is defense-in-depth in action. One hardened control mitigated an entire attack progression. Modern real-world findings show far more LFI than RFI for exactly this reason—PHP's secure-by-default configuration has shifted the vulnerability landscape over the past decade.

**Expected Outcome:** Confirmed that even when exploitable vulnerabilities exist, additional controls at the infrastructure level can prevent full compromise. Both positive (LFI) and properly tested negative (RFI) findings are valuable.

---

# Methodology Summary

| Phase | Objective | Key Technique | Primary Target |
|-------|-----------|----------------|-----------------|
| 1. Burp Suite Mastery | HTTP request interception and manipulation | Proxy configuration; Repeater/Intruder | Mutillidae |
| 2. Injection Attacks | SQL and OS command injection exploitation | Manual techniques + SQLMap automation; shell separator testing | Mutillidae + DVWA |
| 3. Broken Access Control | IDOR and privilege escalation testing | Parameter tampering; cookie inspection; negative testing | Mutillidae |
| 4. XSS & File Inclusion | Client-side and server-side code execution | JavaScript payload injection; path traversal; allow-list bypass | Mutillidae |

---

# Key Takeaway

The methodology progressed from foundational tool mastery (Burp Suite) through single-parameter exploitation (SQLi, Command Injection) to authorization logic (IDOR) to output handling (XSS, LFI). Each phase built on the previous, reinforcing the principle that web application penetration testing is a layered discipline: you cannot test for authorization bypass until you can intercept and modify requests; you cannot exploit IDOR without understanding what "authorization" means in each endpoint's context; you cannot assess risk without testing both working and broken controls.

The engagement also reinforced that real targets rarely match pre-built lab examples exactly. The slide exercise assumed specific parameter names and cookie structures that didn't exist in this Mutillidae build. Adapting methodology to what the target actually presents—identifying equivalent vulnerability surfaces, documenting negative findings when expected exploits don't apply, and resolving infrastructure issues (network configuration, Docker deployment, session state validation) along the way—is the hallmark of professional red-team work.

---
