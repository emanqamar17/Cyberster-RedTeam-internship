# Methodology – Passive Reconnaissance & OSINT

## Overview

This document describes the methodology followed during Week 01 of the Cyberster Red Team Internship. The objective was to collect publicly available information about an authorized training target without directly interacting with its infrastructure.

The methodology follows a structured reconnaissance process commonly used during the information gathering phase of penetration testing.

---

# Reconnaissance Workflow

```text
                     Target Selection
                            │
                            ▼
                Passive Information Gathering
                            │
                            ▼
                 Subdomain Enumeration
                            │
                            ▼
                 Data Consolidation & Validation
                            │
                            ▼
                  Infrastructure Profiling
                            │
                            ▼
                 Technology Fingerprinting
                            │
                            ▼
             Google & GitHub Dorking
                            │
                            ▼
                 Live Asset Verification
                            │
                            ▼
                  Visual Reconnaissance
                            │
                            ▼
                  Documentation & Reporting
```

---

# Phase 1 – Target Identification

## Objective

Identify the authorized target that will be assessed throughout the reconnaissance engagement.

### Target

```
hackthissite.org
```

### Goal

Establish the scope of the assessment before collecting intelligence.

---

# Phase 2 – Passive Information Gathering

## Objective

Collect publicly available information about the target without sending intrusive requests to its infrastructure.

### Activities

- Search public intelligence sources
- Review publicly available records
- Identify external assets
- Gather domain-related information

### Expected Outcome

Build an initial understanding of the organization's external attack surface.

---

# Phase 3 – Subdomain Enumeration

## Objective

Identify subdomains associated with the target domain using multiple passive data sources.

### Tools Used

- Subfinder
- Assetfinder
- Amass
- CRT.sh

### Why Multiple Tools?

Every reconnaissance tool relies on different intelligence sources.

Using multiple tools improves overall coverage and reduces the chance of missing important assets.

### Expected Outcome

A consolidated list of discovered subdomains.

---

# Phase 4 – Data Consolidation & Validation

## Objective

Merge the outputs collected from different reconnaissance tools into a single dataset.

### Activities

- Remove duplicate entries
- Compare results
- Validate discovered assets
- Prepare the final enumeration list

### Expected Outcome

A clean and reliable inventory of discovered subdomains.

---

# Phase 5 – Infrastructure Profiling

## Objective

Collect publicly available information about the infrastructure supporting the target.

### Activities

- WHOIS lookup
- DNS analysis
- Name server identification
- IP information review

### Expected Outcome

A better understanding of hosting infrastructure and network configuration.

---

# Phase 6 – Technology Fingerprinting

## Objective

Identify technologies used by the discovered web applications.

### Tools

- WhatWeb
- Wappalyzer

### Information Collected

- Web server
- Programming language
- Frameworks
- CMS
- JavaScript libraries
- Security headers

### Expected Outcome

Technology stack identification for future testing.

---

# Phase 7 – Google & GitHub Dorking

## Objective

Search publicly indexed information that may expose useful intelligence.

### Google Dorking

Used to identify:

- Public documents
- Login portals
- Sensitive files
- Indexed directories

### GitHub Dorking

Used to identify:

- API keys
- Tokens
- Credentials
- Configuration files
- Source code references

### Expected Outcome

Identify publicly exposed information that may assist future assessments.

---

# Phase 8 – Live Asset Verification

## Objective

Determine which discovered assets are currently active.

### Tools

- HTTPX
- Httprobe

### Activities

- Validate HTTP responses
- Verify HTTPS services
- Identify reachable hosts

### Expected Outcome

A verified list of live web assets.

---

# Phase 9 – Visual Reconnaissance

## Objective

Capture screenshots of discovered web applications for documentation and rapid identification.

### Tool

GoWitness

### Benefits

- Quickly identify applications
- Detect administrative portals
- Differentiate production and development systems
- Improve reporting quality

---

# Phase 10 – Documentation

## Objective

Document every step of the engagement in a clear and repeatable format.

### Documentation Included

- Methodology
- Commands executed
- Technical findings
- Lessons learned
- Screenshots
- Final observations

---

# Methodology Summary

| Phase | Objective |
|--------|-----------|
| Target Selection | Define assessment scope |
| Passive Information Gathering | Collect OSINT |
| Subdomain Enumeration | Discover external assets |
| Data Validation | Remove duplicates and verify results |
| Infrastructure Profiling | Analyze hosting and DNS |
| Technology Fingerprinting | Identify technologies |
| Google & GitHub Dorking | Discover public intelligence |
| Live Asset Verification | Confirm reachable hosts |
| Visual Reconnaissance | Capture screenshots |
| Documentation | Produce structured technical documentation |

---

# Key Takeaway

A structured methodology ensures that reconnaissance activities are systematic, repeatable, and well documented. Combining multiple passive intelligence sources provides a more comprehensive understanding of a target's external attack surface while minimizing the likelihood of overlooking important assets.
