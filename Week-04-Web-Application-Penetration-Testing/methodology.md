# Methodology – Web Application Penetration Testing (OWASP Top 10)

## Overview

This week's engagement followed a progression-based methodology, moving from request-level manipulation (Burp Suite) through injection attacks (SQL and OS command) to authorization testing (IDOR, cookie manipulation) and then client-side and server-side file handling (XSS, LFI, RFI). Each phase built on the tool familiarity and findings of the previous one. The guiding principle was that penetration testing is iterative: technique is adapted to what the target actually exposes rather than forcing pre-planned steps onto an unknown surface.

All testing was performed against intentionally vulnerable platforms (OWASP Mutillidae II on Metasploitable 2, and DVWA) in an isolated, authorized lab environment. Infrastructure was validated before each phase, and session state was re-confirmed after any interruption such as a VM reboot or network change.

---

# Web Application Penetration Testing Workflow
PHASE 1: Request-Level Mastery (Burp Suite)
↓
PHASE 2: Data-Layer and System-Level Attacks (SQL Injection, OS Command Injection)
↓
PHASE 3: Authorization and Access Control (IDOR, Cookie Manipulation)
↓
PHASE 4: Client-Side and File Handling (XSS, LFI, RFI)
↓
PHASE 5: Consolidated Analysis and Risk Summary

---

# Phase 1 – Burp Suite Mastery: The Interceptor

## Objective

Establish working fluency with Burp Suite as an HTTP(S) intercepting proxy, with the ability to pause, inspect, and modify live requests before they reach the server. This phase built the core skill behind all later testing: every request line, header, and body is client-controlled and can be rewritten in transit.

## Proxy Configuration

**Target:** OWASP Mutillidae II (`<TARGET_IP>`)

**Activity:** Started Burp Suite in Temporary Project mode and confirmed the Proxy listener on 127.0.0.1:8080. Configured FoxyProxy in Firefox to route traffic through that listener, then verified interception by loading a plain HTTP site with Intercept enabled and confirming the request paused before forwarding.

**Why it matters:** Without a working proxy, downstream testing (SQL injection, IDOR, XSS payload delivery) has no control point. The proxy is the primary interface between the tester and the target.

**Expected Outcome:** Confirmed that every HTTP request and response is visible and editable before the server processes it or the browser renders it.

## Intercepting and Modifying Requests

**Activity:** Escalated from a harmless header change (User-Agent) to intercepting live registration and login POST requests, modifying the submitted values in transit before forwarding.

**Examples:**
- Modified the User-Agent header on a GET request to the application homepage; the page loaded normally
- Intercepted a registration POST and changed the submitted username before forwarding; the account was created under the modified value
- Intercepted a login POST for the newly created account and changed the submitted credentials in flight to a default administrator pair; the application authenticated the request as the administrator account

**Why it matters:** The server must never trust client-supplied authentication data without server-side verification. The application accepted a default administrator credential pair with no indication of lockout or rate-limiting, a live example of OWASP A07 (Identification and Authentication Failures).

**Expected Outcome:** Confirmed that request modification is the foundational attack surface, and that every downstream technique relies on the same mechanism.

## Repeater and Intruder

**Activity:** Used Repeater to isolate the effect of single parameter changes, comparing response length between a valid and an invalid page parameter. Used Intruder in Sniper mode with a five-entry password list against the login endpoint, marking the password field as the payload position.

**Result:** Every attempt returned HTTP 200 OK, so status codes gave no signal. The correct credential was identified by a response-length outlier (27,335 bytes against a baseline of roughly 26,690) and confirmed by the response body text.

**Why it matters:** When status codes don't vary, response length and content become the detection mechanism. This is the same pattern used in credential-stuffing and default-password attacks against production login forms.

**Expected Outcome:** Confirmed that response-based differentiation can succeed where status-code detection fails.

---

# Phase 2 – Injection Attacks: SQL and OS Command

## Objective

Demonstrate that unvalidated input reaching a database query or an operating system shell can be manipulated to extract unintended data or execute unintended commands. This phase combined manual technique-building (syntax error, then boolean-based logic, then UNION-based extraction) with automated tooling (SQLMap) to cover both the methodology and real-world efficiency.

## Manual SQL Injection (Mutillidae user-info page)

**Target:** The username parameter of the user-info lookup page on OWASP Mutillidae II

**Activity:**
1. Identified the injection point through a normal lookup, captured in Burp and sent to Repeater.
2. Confirmed a SQL syntax error by appending a single quote to the username value.
3. Confirmed boolean-based control of the query: a true condition (`AND '1'='1`) returned full data, and a false condition (`AND '1'='2`) returned an empty response with the connection closed.
4. Enumerated the column count with sequential ORDER BY payloads. Columns 1 through 5 succeeded; ORDER BY 6 returned "Unknown column '6' in 'order clause'", confirming a five-column query.
5. Verified UNION SELECT placeholder values were reflected in the page, mapping columns 2, 3, and 4 to visible output fields.
6. Extracted the current database name, database user, and database version in a single request.

**Why it matters:** The error-based, boolean-based, and UNION-based progression mirrors the escalation path used in real penetration tests and breach investigations. Each technique builds on the previous one, and understanding the manual flow matters when automated tools fail or are blocked.

**Expected Outcome:** The query structure was reverse-engineered, the backing database was identified, and the data structure was mapped for extraction.

## Automated SQL Injection (SQLMap)

**Target:** The same username parameter, with full enumeration and extraction

**Workflow:**
1. Database enumeration to list all accessible databases.
2. Table enumeration within the target application database.
3. Column enumeration within the accounts table.
4. Data extraction from the accounts table.

**Results:**
- Injection confirmed through boolean-based blind, error-based (FLOOR), and time-based blind (SLEEP) techniques, across 290 requests
- Backend fingerprinted as MySQL on an unsupported Linux/Apache/PHP stack, all long past end-of-life
- Seven databases enumerated, including the target application database
- Six tables enumerated within the application database, including accounts, credit_cards, and captured_data
- Five columns enumerated in the accounts table, including an is_admin flag and text-typed password field
- All 19 rows of the accounts table extracted, exposing plaintext passwords for every account

**Critical Finding:** All account passwords are stored in plaintext with no hashing. This is a standalone, severe finding independent of the SQL injection vulnerability, violating OWASP A02:2021 (Cryptographic Failures) and PCI-DSS requirements.

**Why it matters:** SQLMap is standard tooling in real penetration testing (PTES, OWASP Testing Guide). Understanding its output, including injection technique selection and fingerprint accuracy, is essential for production assessments.

**Expected Outcome:** Full compromise of the application database's credential store was confirmed, and the infrastructure weaknesses (end-of-life OS and database versions, over-privileged database account) were documented.

## OS Command Injection (DVWA)

**Target:** DVWA Command Injection module, ping input field

**Activity:**
1. Established a baseline with a valid IP address and confirmed normal ping output.
2. Tested a payload with a space before the command separator. No injected output appeared.
3. Tested the same payload with no space before the separator. Ping output was followed by the output of `whoami`, confirming command execution under the www-data account.
4. Tested `ls` through the same separator. Ping output was followed by a directory listing.

**Why it matters:** Small separator and syntax differences can mask a real vulnerability. The space-before-separator variant produced no visible effect, while the space-free variant succeeded. Retesting with syntax variations is part of disciplined methodology, not repeated guessing.

**Expected Outcome:** Remote command execution confirmed in the web server's privilege context.

---

# Phase 3 – Broken Access Control and IDOR

## Objective

Test whether the application enforces authorization when a client-supplied identifier or role indicator is changed. This phase distinguishes authentication (verifying who the user is) from authorization (verifying what that user may access).

## Insecure Direct Object Reference (IDOR)

**Target:** The blog-viewing page, via its author parameter

**Activity:**
1. Located an applicable IDOR surface on the "View Someone's Blog" page, which selects content through an author parameter. The slide example (a user_id parameter on the user-info page) did not apply to this build, which validates username and password as a matched pair.
2. Captured a request for one author's entries and confirmed the expected blog content was returned.
3. Changed the author parameter to another user's name while authenticated as a different, unrelated account. The other user's private blog entries were returned with no ownership check.

**Why it matters:** IDOR is among the most common bug bounty submission categories because it requires no technical exploit, only parameter substitution against an unverified ownership check.

**Expected Outcome:** Confirmed that access control must be verified per endpoint, not assumed application-wide. The same application enforced credential pairing correctly on one page while trusting an identifier without checks on another.

## Privilege Escalation via Cookie Manipulation

**Target:** Session cookies and page source of the application

**Activity:**
1. Logged in as a non-administrative account.
2. Inspected the Cookie header; only the session identifier was present, with no role cookie.
3. Reviewed the page source and response body for hidden role fields or JavaScript variables. None was found.

**Result (Negative Finding):** This build keeps authorization state entirely server-side in the PHP session and does not expose a role cookie or hidden field for the client to tamper with. This is the correct design pattern and is documented as a tested-and-not-vulnerable result.

**Why it matters:** Testing both working and broken paths is essential for accurate risk assessment. A negative result documents a control that is functioning.

**Expected Outcome:** Confirmed that cookie-based privilege escalation is not exploitable on this target.

---

# Phase 4 – Client-Side Code Execution and File Handling

## Objective

Demonstrate how untrusted input reaching an unsafe sink manifests differently depending on where that sink lives: in the client browser (XSS) or in server-side file operations (LFI, RFI).

## Reflected Cross-Site Scripting (XSS)

**Target:** Reflected XSS on the DNS Lookup tool

**Activity:**
1. Submitted a script payload into the hostname input field.
2. A JavaScript popup executed and displayed the session cookie value.
3. Confirmed that the same execution context could send the cookie to an attacker-controlled endpoint, making the impact session hijacking rather than a benign popup.

**Why it matters:** XSS is often dismissed as "just a popup," but it is full JavaScript execution in the victim's browser. The same reflection mechanism that displays an alert can exfiltrate credentials, hijack sessions, or deliver malware.

**Expected Outcome:** Confirmed a session hijacking vector and demonstrated that XSS impact extends well beyond visual popups.

## Local File Inclusion (LFI)

**Target:** The page parameter, application-wide

**Activity:**
1. Replaced the page parameter with an absolute path to a system file.
2. The server returned the full contents of the file, confirming arbitrary local file read.

**Finding:** The page parameter is used application-wide with no allow-list or path validation, making this an application-wide vulnerability rather than a single vulnerable page. Exposed system account names support further attack planning.

**Why it matters:** LFI can expose application configuration (including database credentials), source code, and system files. It is a common stepping stone to full server compromise.

**Expected Outcome:** Confirmed arbitrary file read capability on the server.

## Remote File Inclusion (RFI)

**Target:** The same page parameter, with a remote URL

**Activity:**
1. Hosted a test file on the attacker machine using a simple local HTTP server.
2. Pointed the page parameter at the hosted file using a remote URL.
3. The server returned a PHP warning that URL file access is disabled in the server configuration, and the include was blocked before any remote content was fetched.

**Result (Negative Finding):** RFI is not exploitable on this target because PHP's `allow_url_include` directive is disabled. The application has a dangerous LFI vulnerability, but infrastructure-level hardening blocks the more severe RFI half.

**Why it matters:** This is defense-in-depth in action. One hardened control stopped an entire attack progression, which is why real-world findings often show LFI without RFI.

**Expected Outcome:** Confirmed that even where an exploitable vulnerability exists, a configuration-level control can prevent full compromise.

---

# Methodology Summary

| Phase | Objective | Key Technique | Primary Target |
|-------|-----------|----------------|-----------------|
| 1. Burp Suite Mastery | HTTP request interception and manipulation | Proxy configuration; Repeater and Intruder | OWASP Mutillidae II |
| 2. Injection Attacks | SQL and OS command injection | Manual technique-building, SQLMap automation, shell separator testing | OWASP Mutillidae II, DVWA |
| 3. Broken Access Control | IDOR and privilege escalation testing | Parameter tampering, cookie inspection, negative testing | OWASP Mutillidae II |
| 4. XSS and File Inclusion | Client-side and server-side code execution | Script payload injection, path traversal, remote inclusion testing | OWASP Mutillidae II |

---

# Key Takeaway

The methodology progressed from foundational tool mastery (Burp Suite) through single-parameter exploitation (SQL injection, command injection) to authorization logic (IDOR) and output handling (XSS, LFI). Each phase built on the previous one. You cannot test authorization bypass until you can intercept and modify requests, you cannot exploit IDOR without understanding what authorization means in each endpoint's context, and you cannot assess risk without testing both working and broken controls.

The engagement also reinforced that real targets rarely match pre-built lab examples exactly. The slide exercises assumed parameter names and cookie structures that did not exist in this build. Adapting the methodology to what the target actually presents, identifying equivalent vulnerability surfaces, documenting negative findings, and resolving infrastructure issues along the way is the core of professional red-team work.

---
