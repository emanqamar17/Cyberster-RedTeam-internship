<p align="center">
  <img src="images/week-04-banner.png" alt="Week 04 Banner" width="100%">
</p>

# Week 04 – Web Application Penetration Testing (OWASP Top 10)

![Status](https://img.shields.io/badge/Status-Complete-brightgreen) ![Week](https://img.shields.io/badge/Week-04-blue) ![Type](https://img.shields.io/badge/Type-Web%20Application%20Testing-orange)

## Overview

This week covered end-to-end web application penetration testing against the OWASP Top 10 vulnerability classes, using industry-standard tooling and methodology. Testing was conducted across two targets—Metasploitable 2 (OWASP Mutillidae II) and DVWA (Docker)—in an isolated lab environment. The engagement demonstrated both the technical mechanics of common web vulnerabilities and the practical red-team workflow of adapting pre-built lab examples to the actual application behavior present in each target.

Four primary tasks were executed: proxy-based HTTP traffic interception and manipulation, SQL injection (manual technique-building followed by automated tooling), operating system command injection, and broken access control (IDOR). Testing also covered client-side attacks (Reflected XSS) and server-side file handling (Local and Remote File Inclusion). The engagement produced eight confirmed, exploitable findings—most critically, a fully extractable plaintext-password database via SQL Injection—alongside three properly tested negative results that confirm specific security controls are functioning correctly.

Beyond the technical exploits themselves, this week reinforced infrastructure troubleshooting, environment validation, and the discipline of testing what actually exists rather than what a slide assumes exists.

> **Target:** Metasploitable 2 (OWASP Mutillidae II) — 192.168.56.101; DVWA (Docker) — 127.0.0.1:8081
> **Environment:** Kali Linux (VirtualBox), Burp Suite Community Edition, SQLMap v1.10.6
> **Engagement Type:** Web Application Penetration Testing — OWASP Top 10

**Key Outcome:** Identified 8 confirmed vulnerabilities including 2 Critical (SQLi with plaintext credential extraction, OS Command Injection) and 3 properly tested negative findings demonstrating working controls.

---

# Learning Objectives

- Configured and operated Burp Suite as an intercepting proxy for live traffic manipulation
- Performed HTTP request/response interception and in-flight modification of headers, credentials, and parameters
- Executed manual SQL injection using error-based, boolean-based, and UNION-based techniques
- Automated SQL injection detection and full-database extraction using SQLMap
- Demonstrated OS command injection via shell separator syntax and command chaining
- Identified and exploited Insecure Direct Object References (IDOR) in application endpoints
- Confirmed Reflected XSS and validated session hijacking attack vectors
- Performed Local File Inclusion (LFI) and tested Remote File Inclusion (RFI) with configuration-level blocking
- Documented both positive findings and properly tested negative results (working security controls)
- Resolved real infrastructure challenges (network adapter configuration, Docker deployment, session state validation)

---

# Skills Developed

- Burp Suite Proxy, Repeater, and Intruder workflow and attack modes
- Manual SQL injection payload construction and DBMS fingerprinting
- SQLMap command-line usage, DBMS detection, and data extraction automation
- OS command injection via shell separators and command chaining
- IDOR surface identification and parameter tampering techniques
- XSS payload crafting and JavaScript execution context awareness
- File inclusion traversal, path manipulation, and protocol handling
- VirtualBox network adapter configuration for isolated lab environments
- Docker image deployment and container networking
- Negative testing methodology and documentation of working controls

---

# Tools Used

| Category | Tools |
|----------|-------|
| Proxies & Interceptors | Burp Suite Community Edition v2026.3.2, FoxyProxy Standard (Firefox) |
| Automation & Scanning | SQLMap v1.10.6 |
| Container & Infrastructure | Docker, Python HTTP server (RFI testing), VirtualBox |
| Targets | Metasploitable 2 (OWASP Mutillidae II v2.1.19), DVWA (vulnerables/web-dvwa Docker image) |
| Attacker OS | Kali Linux 2026.1 (VirtualBox): eth0 Host-only (192.168.56.102), eth1 NAT (10.0.3.15) |

---

# Web Application Penetration Testing Workflow
[1. Burp Suite Mastery]
↓ (Proxy configuration + intercept ON)
[2. HTTP Request Tampering]
↓ (Headers, credentials, parameters)
[3. Repeater & Intruder]
↓ (Manual resend; automated payload testing)

[4. SQL Injection — Manual Phase]
↓ (Syntax errors → Boolean-based logic → UNION SELECT)
[5. SQL Injection — Automated Phase]
↓ (SQLMap DBMS detection + extraction)

[6. OS Command Injection]
↓ (Shell separator syntax testing)

[7. Broken Access Control (IDOR)]
↓ (Parameter substitution; authorization check bypass)
[8. Privilege Escalation Testing]
↓ (Cookie tampering; negative result: server-side session)

[9. Client-Side Code Execution (XSS)]
↓ (Reflected payload → DOM access → hijacking vector)

[10. Server-Side File Handling]
↓ (Local File Inclusion → /etc/passwd)
↓ (Remote File Inclusion → blocked by PHP config)

[11. Consolidated Finding Analysis & Risk Summary]

---

# Results Summary

| Activity | Result |
|----------|-------:|
| **SQL Injection (manual + automated)** | 19 plaintext account records extracted; 7 databases enumerated; 6 tables within owasp10 identified; full credential dump with is_admin flags |
| **Broken Access Control (IDOR)** | author parameter on view-someones-blog.php allows unauthorized blog access; no ownership validation; any authenticated user can read any user's private blog |
| **Reflected Cross-Site Scripting (XSS)** | Session cookie (PHPSESSID) directly accessible via `document.cookie`; JavaScript execution confirmed; full session hijacking vector viable |
| **Local File Inclusion (LFI)** | Application-wide via page parameter with no allow-list; /etc/passwd fully exposed; service account credentials (mysql, postgres, www-data) and msfadmin revealed |
| **OS Command Injection** | Remote command execution confirmed via shell separator (;); directory listing and whoami output returned; www-data privilege context confirmed |
| **Plaintext Password Storage** | **Severity: Critical**; all 19 account passwords stored without hashing; independent of SQL injection vulnerability itself; violates OWASP A02:2021 and PCI-DSS |
| **Excessive Database Privileges** | MySQL application account runs as root@localhost; severe over-privilege enabling full DB compromise |
| **Weak/Default Credentials** | admin/admin accepted without rate-limiting or lockout; no account enumeration protections |
| **Working Controls (Negative Findings)** | 3 properly tested: username-only IDOR rejected on user-info.php; cookie-based privilege escalation impossible (server-side PHP session state); RFI blocked by PHP's allow_url_include=Off configuration |

---

# What I Learned

**Strongest lesson first:** The textbook exploit path often doesn't match the target's actual surface. The slide exercise assumed a `user_id` parameter and a client-exposed `role` cookie on Mutillidae. Neither existed. Instead, I had to identify the actual IDOR vector (the `author` parameter on a different page), confirm that an alternate escalation path (role cookie manipulation) was properly defended against, and document both as valid results. This taught me that template-based hacking fails; real red-teaming means understanding what's in front of you, not just executing pre-planned steps.

Small syntax variations can hide real vulnerabilities. A shell command with a space before the separator (`127.0.0.1 ; whoami`) produced no injected output, while no space succeeded. I almost concluded the target wasn't vulnerable until I retested; the difference in parsing rules is a genuine nuance that separates a real vulnerability from a false negative.

Infrastructure state persists across tasks and VMs. After a reboot, my PHPSESSID was invalid, breaking SQL payloads that had worked moments before. Re-authentication and baseline behavior re-validation became routine discipline, not optional steps.

Burp Suite's visual History view can hide evidence that's clearly there in Raw mode. I thought a modified User-Agent header hadn't been sent because the History list didn't visually flag it—but the Inspector and Raw request tabs showed it clearly. Trusting the tool's underlying data, not its UI defaults, prevented a false negative.

Defense-in-depth works. Mutillidae had a dangerous LFI vulnerability, but PHP's `allow_url_include=Off` configuration blocked the more severe RFI half. This is why modern real-world findings are increasingly LFI without RFI; a single hardened control at the infrastructure level can stop an entire attack progression—and it's why both exploitable and hardened findings belong in a report.

---

# Related Documentation

| Document | Description |
|----------|-------------|
| methodology.md | Detailed methodology for each of the 4 tasks (Burp Suite mastery, Injection attacks, Access Control, XSS/File Inclusion); workflow diagrams per phase |
| commands.md | Real commands executed: Burp Suite configuration, SQLMap payloads with output examples, Docker deployment, shell injection syntax variations |
| findings.md | Executive summary of all 8 confirmed vulnerabilities with OWASP categorization, severity, and location; recommendations for future assessment |
| lessons-learned.md | Technical lessons from the week, challenges encountered and resolved, problem-solving approach, professional skills developed, personal reflection |
| images/README.md | Screenshot evidence index (Figures 4.0–4.33) organized by task and finding type with alt text and technical captions |

---

# Disclaimer

All activity documented in this report was conducted exclusively within an authorized, isolated training laboratory environment. Targets (Metasploitable 2, DVWA) are intentionally vulnerable platforms designed for educational use. No production systems, real data, or unauthorized third-party infrastructure were involved. This engagement is part of the Cyberster Red Team Internship Program (Batch 2026) and is for training purposes only — **NOT FOR DISTRIBUTION**.

---
