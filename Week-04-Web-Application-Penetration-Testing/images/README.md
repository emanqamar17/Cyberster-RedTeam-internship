# Week 04 – Screenshot Evidence

Screenshots are cropped or redacted to remove credentials, session identifiers, IP addresses, and local file paths before being saved to this folder.

| Screenshot File | Report Figure | Description |
|-----------------|---------------|-------------|
| week-04-docker-ps.png | Figure 4.1 | `docker ps` confirming the DVWA container running and mapped to port 8081 |
| week-04-user-agent-modified.png | Figure 4.2 | Modified User-Agent header forwarded to Mutillidae; page loads normally |
| week-04-login-modified-in-flight.png | Figure 4.4 | Login request modified in flight; application authenticated the request as the administrator account (password value redacted) |
| week-04-intruder-length-outlier.png | Figure 4.10 | Intruder attack results; one payload's response length stands out from the baseline of the other attempts |
| week-04-sqli-syntax-error.png | Figure 4.14 | SQL syntax error triggered by a single quote in the username parameter |
| week-04-sqli-boolean-confirmation.png | Figure 4.15 | Boolean-based confirmation: true condition returns full data; false condition returns an empty response with the connection closed |
| week-04-sqli-order-by-6-error.png | Figure 4.17 | ORDER BY 6 triggers "Unknown column" error, confirming a five-column query |
| week-04-sqli-union-column-mapping.png | Figure 4.19 | UNION SELECT 1,2,3,4,5 reflects injected values in the visible output fields |
| week-04-sqli-db-recon.png | Figure 4.20 | Combined UNION SELECT extracting database name, database account, and version (values redacted) |
| week-04-sqlmap-injection-confirmed.png | Figure 4.21a | SQLMap confirms the username parameter is injectable and enumerates seven databases |
| week-04-sqlmap-tables.png | Figure 4.22 | SQLMap enumerates six tables in the application database |
| week-04-cmdi-whoami-www-data.png | Figure 4.26 | `127.0.0.1; whoami` injection confirms command injection; output includes www-data |
| week-04-cookie-page-source-no-role.png | Figure 4.30a | Cookie header and page-source inspection confirm no client-exposed role or privilege indicator |
| week-04-idor-author-parameter.png | Figure 4.28 | Captured request for the blog-viewing page; author parameter identified as the IDOR target |
| week-04-idor-admin-blog-returned.png | Figure 4.29 | Changing the author parameter returns another user's private blog entries; IDOR confirmed (cookie values redacted) |
| week-04-xss-cookie-popup.png | Figure 4.31 | Script payload executes on the DNS Lookup page; popup displays the session cookie value (value redacted) |
| week-04-lfi-etc-passwd.png | Figure 4.32 | Page parameter with an absolute system file path returns the file contents; LFI confirmed (contents redacted to URL bar and first line) |
| week-04-rfi-blocked.png | Figure 4.33 | RFI attempt blocked; PHP's allow_url_include is disabled (attacker address redacted) |

---
