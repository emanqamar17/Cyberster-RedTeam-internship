## Overview

This document captures the technical lessons, challenges, and professional takeaways from Week 4. Each lesson is tied to something that happened during testing, including the failed attempts and environment issues that shaped the outcome. Sensitive values such as credentials, session identifiers, and host addresses have been removed.

---

# Technical Lessons

## 1. The Textbook Exploit Path Did Not Match the Target

The slide exercise assumed a `user_id` parameter on the user-info page and a client-exposed role cookie. Neither existed in this build. The user-info page validated username and password as a matched pair, and the session held no role cookie at all. The applicable IDOR surface turned out to be the author parameter on the blog-viewing page, and the cookie-based escalation path was a tested negative result.

The lesson is that a lab exercise describes one example of a vulnerability class, not the vulnerability itself. Recognizing the underlying mechanism (unverified client-supplied identifier) and locating it on a different page was more valuable than reproducing the slide step by step.

## 2. Small Syntax Differences Can Hide Real Vulnerabilities

The first command injection payload used a space before the semicolon separator. It produced no injected output, and I nearly recorded the parameter as not vulnerable. Removing the space made the injected command execute and return output.

The lesson is that a failed payload is a hypothesis test, not a verdict. Before concluding a target is secure, retest with small variations in spacing, separators, and comment syntax. The same caution applied to SQL comment syntax: a mistyped comment marker broke an otherwise valid ORDER BY payload until it was corrected.

## 3. Reboots Invalidate Session State, and Payloads Fail Silently

After a VM reboot, the browser session identifier changed, and requests that had worked minutes earlier returned empty responses. The payloads themselves were not wrong; the requests were unauthenticated. Diagnosing this required separating two causes at once: a stale session, and a payload typo.

The lesson is to re-authenticate, re-verify a known-good baseline request, and only then retest the payload. Assuming the target changed is the wrong first conclusion.

## 4. Burp's History View Can Hide Evidence

I initially believed a modified User-Agent header had not been sent, because the History list did not visually flag the edited request. Inspecting the raw request in the Inspector showed the header was present.

The lesson is to verify tampered requests in the raw view, not by scanning the summary list. Tools summarize data, and the summary can omit what the underlying request actually contains.

## 5. A Single Hardened Control Can Stop an Attack Chain

The application had a Local File Inclusion vulnerability, which allows reading arbitrary files. Remote File Inclusion, which allows executing attacker-hosted code, would have been far more severe. PHP's `allow_url_include` setting was disabled, so the RFI attempt was blocked before any remote content was fetched.

The lesson is that severity depends on the whole chain, not only on the flaw itself. A negative finding that shows a control blocking the worst outcome is a legitimate and useful result for a report.

---

# Challenges Encountered

- **Docker image pull failed with a DNS resolution error.** The VM's only network adapter was host-only, with no route to the internet. Adding a second NAT adapter restored connectivity without breaking reachability to the lab target.
- **Modified request not visible in Burp's History list.** The edit was present but not flagged in the summary view. The Raw request tab confirmed it.
- **Payloads returned empty responses after a VM reboot.** The session identifier had changed, so requests were unauthenticated. Re-authenticating restored normal behavior.
- **A mistyped SQL comment broke an ORDER BY payload.** The comment marker was missing a space, which invalidated the SQL syntax. Correcting the syntax resolved it.
- **The slide's IDOR and cookie examples did not exist in this build.** The equivalent vulnerability class was located on a different page, and the cookie-based path was documented as a negative result.
- **Command injection with a leading space produced no visible output.** Removing the space made the injection succeed.

---

# Problem-Solving Approach

- **Isolate one variable at a time.** Burp Repeater was used to change a single parameter per request, so each result could be attributed to one change.
- **Establish a baseline before testing.** A valid input was always sent first to confirm normal behavior before a malicious one was tested.
- **Verify environment state before blaming the target.** After reboots and network changes, session state and connectivity were checked before interpreting failed payloads.
- **Treat negative results as data.** Blocked attempts were documented with the control responsible, rather than discarded.
- **Fix infrastructure problems at the root.** The Docker failure was traced to network configuration, not retried blindly.

---

# Professional Skills Developed

- Documenting failed attempts with the same rigor as successful exploits
- Adapting published lab instructions to the actual behavior of a target
- Diagnosing environment problems (network routing, session state) before drawing conclusions about the target
- Communicating severity by describing impact and attack chains, not only the presence of a bug
- Keeping evidence handling disciplined, including redacting credentials and session values from shared documentation

---

# How This Week Supports Future Work

- Burp Suite proficiency carries directly into web application assessments and bug bounty work
- Manual SQL injection understanding makes SQLMap output easier to interpret and verify
- Authorization testing per endpoint is a core skill for API and web application reviews
- Negative-result documentation is a required habit for client-facing reports
- Container and network troubleshooting is routine in lab and production environments alike

---

# Personal Reflection

The most useful part of this week was not any single exploit. It was learning that the first conclusion is often wrong. I expected the lab to follow the slides, and when it did not, the work became more interesting. Retesting a payload with a different spacing, or realizing that a failed request was simply unauthenticated, changed my results more than any tool did. I also came away with a clearer respect for defense-in-depth: a single configuration setting stopped an attack path that the application itself left open.

---

# Key Takeaways

- Lab slides describe a vulnerability class, not a fixed recipe; locate the mechanism on the actual target
- A failed payload is a hypothesis to retest, not proof of security
- Verify session state and connectivity before interpreting results after any environment change
- Negative findings belong in the report, especially when a control blocks the worst outcome
- Retest with small syntax variations before concluding a parameter is not vulnerable
