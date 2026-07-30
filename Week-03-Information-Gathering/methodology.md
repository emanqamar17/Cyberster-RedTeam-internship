# Methodology – Vulnerability Assessment, Initial Exploitation & Web Application Penetration Testing

## Overview

This document describes the methodology followed during Week 03 of the Cyberster Red Team Internship. The objective was to transition from passive information gathering into active enumeration, vulnerability research, and controlled exploitation against authorized lab systems and a designated training platform.

The methodology follows a structured assessment process commonly used during the vulnerability assessment and initial exploitation phases of penetration testing.

---

# Assessment Workflow

```text
                Service Version Collection
                            │
                            ▼
              Web Directory & Parameter Discovery
                            │
                            ▼
                  CVE Mapping & Exploit Research
                            │
                            ▼
                  Exploit Selection & Execution
                            │
                            ▼
                  Exploit Verification
                            │
                            ▼
                 CMS Security Assessment
                            │
                            ▼
                  Documentation & Reporting
```

---

# Phase 1 – Service Version Collection

## Objective

Confirm the exact service versions running on the target before researching any exploits.

### Source

```
Nmap service version detection (carried over from Week 02)
nmap -sV 192.168.56.101
```

### Goal

Establish an accurate baseline of open ports, running services, and software versions to guide exploit research in later phases.

---

# Phase 2 – Web Directory & Parameter Discovery

## Objective

Discover hidden directories, sensitive files, and undocumented URL parameters on the authorized training platform that are not linked from the main application interface.

### Target

```
hackthissite.org
```

### Tools Used

- FFUF
- Dirsearch
- Gobuster
- Arjun
- ParamSpider

### Why Multiple Tools?

Each tool relies on a different discovery technique — active fuzzing, extension-aware brute-forcing, and archived URL mining. Using multiple tools improves coverage and reduces the chance of missing an entry point that a single tool might overlook.

### Expected Outcome

A consolidated list of hidden directories, sensitive files, and hidden parameters representing potential attack surfaces.

---

# Phase 3 – CVE Mapping & Exploit Research

## Objective

Map the service versions identified in Phase 1 to known vulnerabilities and confirm that a working public exploit exists.

### Activities

- Search Searchsploit using the service name and version.
- Confirm CVE identifiers and CVSS severity on NVD.
- Cross-reference the same service version or CVE ID on Exploit-DB.

### Why This Order?

Searchsploit provides a fast offline indication of exploit availability. NVD confirms the vulnerability is officially documented and rates its severity. Exploit-DB then verifies a working public exploit exists and provides its reference ID.

### Expected Outcome

A prioritized list of vulnerabilities capable of providing an initial foothold, focusing on remote code execution over lower-impact issues.

---

# Phase 4 – Exploit Selection & Execution

## Objective

Test only the exploits that match confirmed service versions, using the Metasploit Framework in a controlled manner.

### Standard Execution Sequence

```
search <service_name>
use <exploit_module>
set RHOSTS <target_ip>
set LHOST <attacker_ip>
set payload <payload_type>
run
```

### Expected Outcome

Either a confirmed shell session or a clearly documented failed attempt, in both cases supported by verification rather than assumption.

---

# Phase 5 – Exploit Verification

## Objective

Confirm the true outcome of every exploitation attempt rather than relying on the module's reported status alone.

### Activities

- Use `check` and `info` to review module state and target compatibility.
- Manually confirm expected backdoor ports using Nmap (e.g. `nmap -p 6200 <target>`).
- On successful shells, run `whoami`, `id`, and `uname -a` to confirm access level and target identity.

### Why Verify Before Concluding?

A module reporting "success" does not always mean a usable session was created. Matching a vulnerable service version to an exploit does not guarantee successful exploitation — the target environment must still be validated independently.

### Expected Outcome

An accurate, evidence-backed record of which exploits succeeded and which failed, including the reason for failure where applicable.

---

# Phase 6 – CMS Security Assessment

## Objective

Assess CMS-specific weaknesses using dedicated scanning tools against Docker-deployed WordPress and Joomla instances.

### WordPress (WPScan)

- Baseline scan to identify version, server technology, and exposed files.
- User enumeration (`--enumerate u`) to identify valid usernames.
- Vulnerable plugin enumeration (`--enumerate vp`).

### Joomla (JoomScan)

- Baseline scan to confirm Joomla detection and version.
- Component and plugin enumeration.
- Supplementary directory enumeration with Gobuster against `/components/`.

### Why Docker?

Docker allowed both vulnerable CMS environments to be deployed and torn down in isolation from the host system, without affecting other lab components.

### Expected Outcome

A documented set of CMS-specific misconfigurations, exposed files, and enumerable accounts for each platform.

---

# Phase 7 – Documentation

## Objective

Document every step of the engagement in a clear and repeatable format.

### Documentation Included

- Methodology
- Commands executed
- Technical findings
- Challenges encountered and resolutions
- Screenshots
- Successful and failed exploitation attempts

---

# Methodology Summary

| Phase | Objective |
|--------|-----------|
| Service Version Collection | Establish an accurate service baseline |
| Web Directory & Parameter Discovery | Identify hidden attack surfaces |
| CVE Mapping & Exploit Research | Match services to known, working exploits |
| Exploit Selection & Execution | Test only version-matched exploits |
| Exploit Verification | Confirm true outcome of each attempt |
| CMS Security Assessment | Identify WordPress/Joomla-specific weaknesses |
| Documentation | Produce structured technical documentation |

---

# Key Takeaway

A structured methodology ensures that vulnerability assessment and exploitation activities are systematic, repeatable, and well documented. Confirming a service version matches an available exploit is only the starting point — independent verification of every outcome, successful or failed, is what separates a reliable assessment from an assumed one.
