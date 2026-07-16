# Findings – Passive Reconnaissance & OSINT

## Executive Summary

During the reconnaissance phase, publicly available information was collected from multiple passive intelligence sources to build an initial understanding of the target's external attack surface.

The assessment successfully identified active web assets, supporting infrastructure, DNS information, technologies, and publicly exposed resources without performing intrusive interactions with the target environment.

---

# Assessment Statistics

| Metric | Result |
|---------|-------:|
| Target Domain | hackthissite.org |
| Passive Intelligence Sources | 5 |
| Unique Subdomains Identified | 68 |
| Live Web Assets | 34 |
| Screenshots Captured | 18 |
| Internship Tasks Completed | 4 |

---

# Attack Surface Overview

The reconnaissance process revealed multiple externally accessible assets associated with the target domain.

These assets included:

- Primary web applications
- Additional subdomains
- Development-related services
- Supporting infrastructure
- Publicly accessible web resources

Mapping the attack surface provides a foundation for future vulnerability assessment activities.

---

# Infrastructure Findings

Infrastructure profiling identified useful publicly available information regarding the target environment.

Information gathered included:

- Domain registration details
- Name servers
- DNS records
- Hosting-related information
- Public IP information

This information assists in understanding how the target environment is structured.

---

# Technology Stack Observations

Technology fingerprinting identified several technologies used across the target environment.

Examples include:

- Web server technologies
- Programming languages
- Frameworks
- Content Management Systems (where applicable)
- JavaScript libraries
- Security-related HTTP headers

Understanding the technology stack helps guide future vulnerability testing by identifying technologies that may have known weaknesses or configuration issues.

---

# Subdomain Enumeration Results

Subdomain enumeration was performed using multiple passive intelligence sources.

### Sources Used

- Subfinder
- Assetfinder
- Amass
- CRT.sh
- Public OSINT datasets

Combining multiple sources improved coverage and reduced the likelihood of missing externally exposed assets.

---

# Live Asset Verification

After enumeration, discovered hosts were validated using HTTPX and Httprobe.

This process identified:

- Active HTTP services
- Active HTTPS services
- Reachable web applications
- Accessible targets for future testing

Inactive assets were excluded from subsequent analysis to improve efficiency.

---

# Google Dorking & GitHub Dorking

Public search techniques were used to identify information exposed through search engines and public repositories.

The assessment focused on locating:

- Public documents
- Login portals
- Configuration files
- API references
- Public code repositories
- Potential credential exposure

No unauthorized access attempts were performed.

---

# Visual Reconnaissance

GoWitness was used to capture screenshots of active web applications.

Benefits included:

- Quick identification of applications
- Recognition of administrative interfaces
- Identification of development or staging environments
- Improved documentation quality

---

# Security Observations

The reconnaissance process highlighted several important security considerations:

- Publicly exposed assets increase the external attack surface.
- Multiple technologies may require different testing methodologies.
- Publicly indexed information can reveal valuable intelligence.
- Passive reconnaissance provides meaningful information before any active interaction with the target.

---

# Recommendations for Future Assessment

Based on the reconnaissance phase, the following activities would logically follow:

1. Active reconnaissance
2. Directory and file enumeration
3. Service enumeration
4. Vulnerability assessment
5. Web application fingerprinting
6. Authentication testing
7. OWASP Top 10 testing
8. Manual penetration testing

---

# Conclusion

The passive reconnaissance phase successfully established a comprehensive understanding of the target's external presence.

By combining multiple OSINT sources and validation techniques, a structured inventory of publicly accessible assets was created. This information forms the basis for future phases of the penetration testing lifecycle, including active reconnaissance and vulnerability assessment.
