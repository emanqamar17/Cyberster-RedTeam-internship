# Week 04 – Screenshot Evidence

| Screenshot | Description |
|------------|--------------|
| Figure 4.0 | `ip a` output confirming dual-adapter configuration and successful internet connectivity test |
| Figure 4.1 | `docker ps` confirming the DVWA container running and mapped to port 8081 |
| Figure 4.2 | Modified User-Agent header forwarded to Mutillidae; page loads normally |
| Figure 4.3 | Intercepted registration request with username altered in transit; account created under the modified value |
| Figure 4.4 | Login request modified in flight to default administrator credentials; application authenticated the request as the administrator account |
| Figure 4.5 | Intercepted request sent to Repeater |
| Figure 4.6 | Repeater response-length comparison between an invalid and a valid page parameter |
| Figure 4.7 | Login request loaded into Intruder |
| Figure 4.8 | Intruder Positions tab with password value marked as the single payload position, Sniper attack type |
| Figure 4.9 | Intruder Payloads tab with a five-entry simple list loaded |
| Figure 4.10 | Intruder attack results; one payload's response length stands out from the baseline of the other attempts |
| Figure 4.11 | Response body for the successful payload showing the logged-in state |
| Figure 4.12 | Response body for a failed payload showing the "Not Logged In" message, confirming the length signal was reliable |
| Figure 4.13 | Captured user-info request with username and password parameters, sent to Repeater |
| Figure 4.14 | SQL syntax error triggered by a single quote in the username parameter |
| Figure 4.15 | Boolean-based confirmation: true condition returns full data; false condition returns an empty response with the connection closed |
| Figure 4.16a | Sequential ORDER BY testing, columns 1 through 5 succeeding without error |
| Figure 4.16b | Sequential ORDER BY testing, columns 1 through 5 succeeding without error (continued) |
| Figure 4.17 | ORDER BY 6 triggers "Unknown column" error, confirming a five-column query |
| Figure 4.18 | ORDER BY 5 re-verified successful after VM reboot and fresh authentication |
| Figure 4.19 | UNION SELECT 1,2,3,4,5 reflects injected values in the visible output fields |
| Figure 4.20 | Combined UNION SELECT extracting database name, database account, and version (values redacted in repository copy) |
| Figure 4.21a | SQLMap confirms the username parameter is injectable and enumerates seven databases |
| Figure 4.21b | SQLMap confirms the username parameter is injectable and enumerates seven databases (continued) |
| Figure 4.22 | SQLMap enumerates six tables in the application database |
| Figure 4.23a | SQLMap enumerates five columns in the accounts table |
| Figure 4.23b | SQLMap enumerates five columns in the accounts table (continued) |
| Figure 4.24a | SQLMap dumps all 19 account rows (credential values redacted in repository copy) |
| Figure 4.24b | SQLMap dumps all 19 account rows (credential values redacted in repository copy, continued) |
| Figure 4.25 | Baseline ping output for 127.0.0.1 |
| Figure 4.26 | `127.0.0.1; whoami` injection confirms command injection; output includes www-data |
| Figure 4.27 | `127.0.0.1; ls` returns a directory listing, confirming broader command execution |
| Figure 4.28 | Captured request for the blog-viewing page; author parameter identified as the IDOR target |
| Figure 4.29 | Changing the author parameter returns another user's private blog entries; IDOR confirmed |
| Figure 4.30a | Cookie header and page-source inspection confirm no client-exposed role or privilege indicator |
| Figure 4.30b | Cookie header and page-source inspection confirm no client-exposed role or privilege indicator (continued) |
| Figure 4.31 | Script payload executes on the DNS Lookup page; popup displays the session cookie value (value redacted) |
| Figure 4.32 | Page parameter with an absolute system file path returns the file contents; LFI confirmed (contents redacted) |
| Figure 4.33 | RFI attempt blocked; PHP's allow_url_include is disabled |

---


- Figure 4.0: adapter IP addresses
- Figure 4.32: system account listing
- Any figure showing a terminal prompt with a local username or home path
