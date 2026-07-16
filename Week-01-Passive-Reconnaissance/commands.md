# Commands Used – Passive Reconnaissance & OSINT

## Overview

This document contains the primary commands executed during the passive reconnaissance phase of the engagement. Each command is accompanied by its purpose and expected outcome to provide context and improve reproducibility.

---

# 1. Subfinder

## Purpose

Subfinder is a passive subdomain enumeration tool that collects subdomains from multiple public sources.

### Command

```bash
subfinder -d hackthissite.org
```

### Expected Output

- List of discovered subdomains
- Passive enumeration results
- No direct interaction with the target

### Why It Was Used

Subfinder provides fast and reliable passive enumeration using multiple OSINT sources.

---

# 2. Assetfinder

## Purpose

Identify additional subdomains from publicly available datasets.

### Command

```bash
assetfinder --subs-only hackthissite.org
```

### Expected Output

Additional subdomains that may not appear in Subfinder.

### Why It Was Used

Different tools use different data sources, improving overall coverage.

---

# 3. Amass

## Purpose

Perform passive subdomain enumeration using extensive OSINT sources.

### Command

```bash
amass enum -passive -d hackthissite.org
```

### Expected Output

Large list of discovered subdomains collected from numerous intelligence sources.

### Why It Was Used

Amass generally discovers assets that simpler tools may miss.

---

# 4. HTTPX

## Purpose

Verify which discovered hosts are currently online.

### Command

```bash
httpx -l subdomains.txt
```

### Expected Output

List of live HTTP and HTTPS services.

### Why It Was Used

Not every discovered subdomain is active. HTTPX filters the results to reachable web services.

---

# 5. Httprobe

## Purpose

Validate active web servers.

### Command

```bash
cat subdomains.txt | httprobe
```

### Expected Output

Reachable HTTP and HTTPS endpoints.

### Why It Was Used

Provides an additional validation step for discovered assets.

---

# 6. WhatWeb

## Purpose

Identify technologies running on target websites.

### Command

```bash
whatweb https://hackthissite.org
```

### Expected Output

Technology stack including:

- Web server
- Framework
- Programming language
- CMS
- Security headers

### Why It Was Used

Technology identification helps prepare for future vulnerability assessments.

---

# 7. WHOIS

## Purpose

Collect publicly available registration information.

### Command

```bash
whois hackthissite.org
```

### Expected Output

- Domain registrar
- Registration details
- Name servers
- Important dates

### Why It Was Used

WHOIS provides infrastructure intelligence useful during reconnaissance.

---

# 8. DIG

## Purpose

Query DNS records.

### Command

```bash
dig hackthissite.org
```

### Expected Output

DNS records including:

- A
- MX
- NS
- TXT

### Why It Was Used

DNS information assists in understanding network infrastructure.

---

# 9. NSLOOKUP

## Purpose

Verify DNS resolution.

### Command

```bash
nslookup hackthissite.org
```

### Expected Output

Resolved IP addresses and DNS information.

### Why It Was Used

Provides a quick method for validating DNS responses.

---

# Command Summary

| Tool | Purpose |
|------|---------|
| Subfinder | Passive subdomain enumeration |
| Assetfinder | Additional asset discovery |
| Amass | Comprehensive passive enumeration |
| HTTPX | Live host verification |
| Httprobe | Web service validation |
| WhatWeb | Technology fingerprinting |
| WHOIS | Infrastructure profiling |
| DIG | DNS analysis |
| NSLOOKUP | DNS validation |

---

# Key Takeaway

Using multiple reconnaissance tools provides broader coverage because each tool relies on different public intelligence sources. Combining their outputs results in a more complete and reliable view of the target's external attack surface.
